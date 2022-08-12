---
title: A Smarter Promise.all() for JavaScript (ES2015)
date: 2015-10-20 18:03:51
tags:
  - es6
  - javascript
  - promises
  - tutorial
  - hints
  - example
  - parallelisation
---

Quite often when writing promise-based logic flow I'll need to use a `Promise.all()`. What does `Promise.all()` do you ask?

According to [MDN](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Promise/all):

> The Promise.all(iterable) method returns a promise that resolves when all of the promises in the iterable argument have resolved, or rejects with the reason of the first passed promise that rejects.

Let's build an example that makes use of this; we want to `GET` a load of web pages, find the text content of the `<h1>` on each page and return them as an array once **all** of the requests have finished.

### A simple `Promise.all` example

We're going to be using the excellent [xray](https://github.com/lapwinglabs/x-ray) library to make the requests & parse the HTML. The code for this example is available on github [promise-sequence-example](https://github.com/BBB/promise-sequence-example).

```javascript
import Xray from 'x-ray';

const xray = Xray();

const urlsToGet = ['medium.com', 'twitter.com', 'facebook.com'];

const urlPromises = urlsToGet.map(url => {
  console.log('\tStarting', url);
  return xrayPromise(`http://${url}`, 'h1')
    .then(result => {
      console.log('\tFinished', url, '-', JSON.stringify(result));
      return result;
    })
    .catch(err => {
      // We want errors in this dataset to fail silently
      // this is because some sites may only accept https
      // or perhaps be down.
      // Normally you would want to handle this
      return null;
    });
});

Promise.all(urlPromises)
  .then(console.log.bind(console))
  .catch(err => console.error(`ERROR: ${err.message}\n${err.stack}`));

//
// Helpers
//

// Wrap the standard xray function so that it returns a promise
function xrayPromise(...args) {
  return new Promise((resolve, reject) => {
    xray(...args)((err, obj) => {
      if (err) {
        return reject(err);
      }
      return resolve(obj);
    });
  });
}
```

The result:

```javascript
['Move thinking forward.', 'Welcome to Twitter.', 'Facebook logo'];
```

This serves us very well for a few URLs, but what happens when we try to do this over a much larger set, let's say 1000?

### Fetching the dataset

Let's pull down the top 1m websites as ranked by [alexa](http://www.alexa.com/topsites) and filter them down to the top 1000.

```bash
curl -sv -O http://s3.amazonaws.com/alexa-static/top-1m.csv.zip; \
  unzip -q -o top-1m.csv.zip top-1m.csv; \
  head -1000 top-1m.csv | \
  cut -d, -f2 | \
  cut -d/ -f1 > topsites.txt && \
  rm top-1m.csv.zip top-1m.csv
```

### Running the Example with our Large Dataset

We need to make a few modifications to how we get the source array `urlsToGet`.

**Warning: running the below may cause you computer to run out of CPU/ memory and freeze**

```javascript
import fs from 'fs';

...

const urlsToGet = fs.readFileSync('./topsites.txt').toString('utf8').split(/\r?\n|\r/);

...
```

When I run this I get an error raised from xray.

```javascript
var $ = html.html ? html : cheerio.load(html);
            ^
TypeError: Cannot read property 'html' of null
```

This is caused by the process running out of memory.

### A solution!

What if instead of attempting to do all of the requests and parsing at the same time we split the processing up into smaller chunks?

```javascript
// split the array into smaller arrays,
// process each of the smaller arrays before moving
// onto the next
function promiseSeq(arr, predicate, consecutive = 10) {
  return chunkArray(arr, consecutive).reduce((prom, items, ix) => {
    // wait for the previous Promise.all() to resolve
    return prom.then(allResults => {
      console.log('\nSET', ix);
      return Promise.all(
        // then we build up the next set of simultaneous promises
        items.map(item => {
          // call the processing function
          return predicate(item, ix);
        })
      ).then(results => {
        // then push the results into the collected array
        return allResults.concat(results);
      });
    });
  }, Promise.resolve([]));

  function chunkArray(startArray, chunkSize) {
    let j = -1;
    return startArray.reduce((arr, item, ix) => {
      j += ix % chunkSize === 0 ? 1 : 0;
      arr[j] = [...(arr[j] || []), item];
      return arr;
    }, []);
  }
}
```

### Putting it all together

```javascript
import Xray from 'x-ray';
import fs from 'fs';

const xray = Xray();

const urlsToGet = fs
  .readFileSync('./topsites.txt')
  .toString('utf8')
  .split(/\r?\n|\r/);

const urlPromiseSequence = promiseSeq(urlsToGet, (url, ix) => {
  console.log('\tStarting', url);
  return xrayPromise(`http://${url}`, 'h1')
    .then(result => {
      console.log('\tFinished', url, '-', JSON.stringify(result));
      return result;
    })
    .catch(err => {
      // We want errors in this dataset to fail silently
      // this is because some sites may only accept https
      // or perhaps be down.
      // Normally you would want to handle this
      return null;
    });
});

urlPromiseSequence
  .then(console.log.bind(console))
  .catch(err => console.error(`ERROR: ${err.message}\n${err.stack}`));

//
// Helpers
//

// Wrap the standard xray function so that it returns a promise
function xrayPromise(...args) {
  return new Promise((resolve, reject) => {
    xray(...args)((err, obj) => {
      if (err) {
        return reject(err);
      }
      return resolve(obj);
    });
  });
}

// split the array into smaller arrays,
// process each of the smaller arrays before moving
// onto the next
function promiseSeq(arr, predicate, consecutive = 10) {
  return chunkArray(arr, consecutive).reduce((prom, items, ix) => {
    // wait for the previous Promise.all() to resolve
    return prom.then(allResults => {
      console.log('\nSET', ix);
      return Promise.all(
        // then we build up the next set of simultaneous promises
        items.map(item => {
          // call the processing function
          return predicate(item, ix);
        })
      ).then(results => {
        // then push the results into the collected array
        return allResults.concat(results);
      });
    });
  }, Promise.resolve([]));

  function chunkArray(startArray, chunkSize) {
    let j = -1;
    return startArray.reduce((arr, item, ix) => {
      j += ix % chunkSize === 0 ? 1 : 0;
      arr[j] = [...(arr[j] || []), item];
      return arr;
    }, []);
  }
}
```

We've now broken the 1000 requests into manageable chunks of 10 parallel requests. This will take quite a while to complete, but it won't have the issues with maxing out system resources that the previous example had.

Currently this will do 10 requests at a time. If we wanted `promiseSeq` to behave the same as `Promise.all` we could change the `consecutive` parameter to be the same length as the array of URLs.

Similarly we could decrease the `consecutive` parameter on slower systems, and increase on faster.

The code for this example is available on github [promise-sequence-example](https://github.com/BBB/promise-sequence-example)
