---
title: Credit Card Scanning in React Native
date: 2015-10-07 13:53:17
tags:
  - react
  - javascript
  - open-source
  - react-native
  - card.io
  - iOS
---

I'm happy to announce my first react-native package; `react-native-card.io`. You can view the project on [GitHub](https://github.com/BBB/react-native-card.io), there is also an [Example iOS Project](https://github.com/BBB/react-native-card.io-example) to get you started. Android support is planned, if you'd like to help me implement it, shoot me an email on [ollie@relph.me](mailto:ollie@relph.me) or create a PR.

`react-native-card.io` adds javascript bindings for [card.io](http://card.io/) an open-source project created by [PayPal](http://paypal.com/) and used by [Uber](http://uber.com), [TaskRabbit](http://taskrabbit.com/) and [GrubHub](http://www.grubhub.com/) amongst others.

### Using the Card Scanner

Install the cli tool for initialising react-native projects

```bash
npm i react-native-cli -g
```

Run the react-native setup

```bash
react-native init CardWallet && cd CardWallet
```

Once this is complete we can remove the android code folder as we're not going to be using it.

```bash
rm -rf android index.android.js
```

**At this point you should open the project in XCode and test that it builds ok**

From within the project folder you should run:

```bash
npm i react-native-card.io --save
```

Because `react-native-card.io` contains native code, we need to add it to our XCode project.

Right click on the project and "Add files to ..."
![Add Files to Project](__GHOST_URL__/blog/content/images/2015/10/add-files-to-project.png)

Navigate to your `node_modules` folder and select the Card.io project.
![Select the Card.io XCode Project](__GHOST_URL__/blog/content/images/2015/10/select-the-card-io-project.png)

Add the binary to the Link build step
![Select the Card.io Binary](__GHOST_URL__/blog/content/images/2015/10/select-the-card-io-binary.png)

Add the `-lc++` flag to "Other Linker Flags"
![Add linker flags](__GHOST_URL__/blog/content/images/2015/10/add-linker-flags.png)

Require the component in your view

```javascript
import CardIO from "react-native-card.io";
```

Add it to your render method, don't forget to handle the `onSuccess` and `onFailure` callbacks.

```javascript
<CardIO
  style={{
    flex: 1,
    backgroundColor: "black",
  }}
  onSuccess={(cardInfo) => console.log(cardInfo)}
  onFailure={(err) => console.error(err)}
/>
```

Thats it!
