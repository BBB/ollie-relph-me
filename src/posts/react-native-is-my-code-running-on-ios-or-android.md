---
title: React Native: Is my code running on iOS or Android?
date: 2015-10-17 17:15:48
tags:
  - react-native
  - hints
  - Android
  - iOS
---

React-native `v0.12.0` now comes with support for Android out-of-the-box. 

A common issue when writing cross platform (a.k.a isomorphic) code is determining which platform your code is being executed on.

Luckily there's a simple way:

```javascript
import { Platform } from 'react-native';

if ( Platform.OS === 'ios' ) {
  // iOS specific code here
} else {
  // Android specific code here
}

```

