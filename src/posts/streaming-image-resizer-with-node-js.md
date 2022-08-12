---
title: Streaming Image Resizer with node.js
date: 2016-02-26 13:13:45
tags:
  - node.js
  - streams
  - images
  - resizing
  - javascript
---

For this demo I'm going to be taking some wonderful stock images from [Unsplash](https://unsplash.com/), resizing them into multiple sizes and then finally saving them to disk (though you could also easily upload them to amazon s3), all with the power of [node.js streams](https://nodejs.org/api/stream.html).

We'll be using the awesome [sharp](https://github.com/lovell/sharp) to handle the resizing of images. Sharp means that we don't need to install heavy dependencies like [graphicsmagick](http://www.graphicsmagick.org/)/ [ImageMagick](http://www.imagemagick.org/script/index.php), it also provides a streaming interface, that means we don't need to hit the filesystem when transforming our images. So let's start the project by installing sharp!.

Follow the installation instructions for your OS in the [sharp docs](http://sharp.dimens.io/en/stable/install/)

> Note: this code was written to run on node `v4.2.1`

```bash
npm init
touch index.js
npm i --save sharp
```

Now open `index.js` in your favourite text editor.

The first thing we're going to need is take a uri, create a download stream and pipe it to a writeable stream:

```javascript
// Import both http & https for handling different uris
var http = require('http');
var https = require('https');
// in order to write to the filesystem we need the `fs` lib
var fs = require('fs');

var imageUri =
  'https://images.unsplash.com/photo-1427805371062-cacdd21273f1?ixlib=rb-0.3.5&q=80&fm=jpg&crop=entropy&s=7bd7472930019681f251b16e76e05595';

// determine wether we need to use `http` or `https` libs
var httpLib = http;
if (/^https/.test(imageUri)) {
  httpLib = https;
}
// begin reading the image
httpLib.get(imageUri, function(downloadStream) {
  var writeStream = fs.createWriteStream('./output.jpg');
  downloadStream.pipe(writeStream);
  downloadStream.on('end', () => {
    console.log('downloadStream', 'END');
  });
  writeStream.on('error', err => {
    console.log('writeStream', err);
  });
  downloadStream.on('error', err => {
    console.log('downloadStream', err);
  });
});
```

If you run the script with `node index.js`, you'll see that the image will be gradually be saved to `output.jpg` while it is downloaded. The download & save are both complete once you see `downloadStream END` logged to the console.

### Adding the streaming resizer

We now need to insert the `sharp` streaming processor in the stream pipeline.

```javascript
// import the lib
var sharp = require('sharp');

// create the resize transform
var resizeTransform = sharp()
  .resize(300, 300)
  .max();

downloadStream.pipe(resizeTransform).pipe(writeStream);
```

Putting this together gives us:

```javascript
// Import both http & https for handling different uris
var http = require('http');
var https = require('https');
// in order to write to the filesystem we need the `fs` lib
var fs = require('fs');
// import the lib
var sharp = require('sharp');

// create the resize transform
var resizeTransform = sharp()
  .resize(300, 300)
  .max();

var imageUri =
  'https://images.unsplash.com/photo-1427805371062-cacdd21273f1?ixlib=rb-0.3.5&q=80&fm=jpg&crop=entropy&s=7bd7472930019681f251b16e76e05595';

// determine wether we need to use `http` or `https` libs
var httpLib = http;
if (/^https/.test(imageUri)) {
  httpLib = https;
}
// begin reading the image
httpLib.get(imageUri, function(downloadStream) {
  var writeStream = fs.createWriteStream('./output-300x300.jpg');
  downloadStream.pipe(resizeTransform).pipe(writeStream);
  downloadStream.on('end', () => {
    console.log('downloadStream', 'END');
  });
  writeStream.on('error', err => {
    console.log('writeStream', err);
  });
  downloadStream.on('error', err => {
    console.log('downloadStream', err);
  });
  resizeTransform.on('error', err => {
    console.log('resizeTransform', err);
  });
});
```

### Making it useful

Let's wrap this into a function that returns a `promise` as it's primary interface.

```javascript
// Import both http & https for handling different uris
var http = require('http');
var https = require('https');
// in order to write to the filesystem we need the `fs` lib
var fs = require('fs');
// import the lib
var sharp = require('sharp');

var imageUri =
  'https://images.unsplash.com/photo-1427805371062-cacdd21273f1?ixlib=rb-0.3.5&q=80&fm=jpg&crop=entropy&s=7bd7472930019681f251b16e76e05595';

resizeImage(imageUri, 300, 300)
  .then(thumbnailPath => console.log('DONE', thumbnailPath))
  .catch(err => console.log(err));

function resizeImage(imageUri, width, height) {
  // create the resize transform
  var resizeTransform = sharp()
    .resize(width, height)
    .max();
  return new Promise((resolve, reject) => {
    // determine wether we need to use `http` or `https` libs
    var httpLib = http;
    if (/^https/.test(imageUri)) {
      httpLib = https;
    }
    // begin reading the image
    httpLib.get(imageUri, function(downloadStream) {
      var outPath = `./output-${width}x${height}.jpg`;
      var writeStream = fs.createWriteStream(outPath);
      downloadStream.pipe(resizeTransform).pipe(writeStream);
      downloadStream.on('end', () => resolve(outPath));
      writeStream.on('error', reject);
      downloadStream.on('error', reject);
      resizeTransform.on('error', reject);
    });
  });
}
```

Or perhaps we could pass an array of sizes through and download the image once, and resize it and save it for each size.

```javascript
// Import both http & https for handling different uris
var http = require('http');
var https = require('https');
// in order to write to the filesystem we need the `fs` lib
var fs = require('fs');
// import the lib
var sharp = require('sharp');

var imageUri =
  'https://images.unsplash.com/photo-1427805371062-cacdd21273f1?ixlib=rb-0.3.5&q=80&fm=jpg&crop=entropy&s=7bd7472930019681f251b16e76e05595';

resizeImage(imageUri, [
  [300, 300],
  [600, 450]
])
  .then(thumbnailPaths => console.log('DONE', thumbnailPaths))
  .catch(err => console.log(err));

function resizeImage(imageUri, sizes) {
  return new Promise((resolve, reject) => {
    // determine wether we need to use `http` or `https` libs
    var httpLib = http;
    if (/^https/.test(imageUri)) {
      httpLib = https;
    }
    // begin reading the image
    httpLib.get(imageUri, function(downloadStream) {
      downloadStream.on('error', reject);
      Promise.all(sizes.map(size => resizeAndSave(downloadStream, size)))
        .then(resolve)
        .catch(reject);
    });
  });

  function resizeAndSave(downloadStream, size) {
    // create the resize transform
    var resizeTransform = sharp()
      .resize(size[0], size[1])
      .max();
    return new Promise((resolve, reject) => {
      var outPath = `./output-${size[0]}x${size[1]}.jpg`;
      console.log('WRITING', outPath);
      var writeStream = fs.createWriteStream(outPath);
      downloadStream.pipe(resizeTransform).pipe(writeStream);
      downloadStream.on('end', () => resolve(outPath));
      writeStream.on('error', reject);
      resizeTransform.on('error', reject);
    });
  }
}
```

I'll leave implementing Amazon s3 streaming, handling different image formats & adding extra `sharp` transforms as an exercise to the reader.
