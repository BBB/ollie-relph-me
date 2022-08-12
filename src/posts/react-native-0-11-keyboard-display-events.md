---
title: React Native 0.11+ Keyboard Display Events
date: 2015-10-07 14:25:03
tags:
  - react-native
  - hints
  - react

---

In React Native 0.11.0-rc we no longer need a package like [react-native-keyboardevents](https://github.com/johanneslumpe/react-native-keyboardevents) we can simply use:

```javascript
import { DeviceEventEmitter } from 'react-native';

DeviceEventEmitter.addListener('keyboardWillShow', (e) => {
  // Use e.endCoordinates.height
  // to set your view's marginBottom
  // ...
});

DeviceEventEmitter.addListener('keyboardWillHide', (e) => {
  // ...
});
```
