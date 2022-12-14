---
title: "Testing Accessibility with React and React-Native"
date: 2020-07-08 17:00:57
---

[testing-a11y](https://github.com/BBB/testing-a11y) simplifies switching your app between test and accessible modes, stops the temptation of hard-coding selectors for IDs and makes the whole thing much more simple.

Here's why it exists:

You have a component that you'd like to write integration tests for in the simulator using something like appium or detox.

You've added accessibility, and platform specific code. It's alot to remember for each place you'd like an ID, or just some a11y text.

```typescript
import * as React from "react";
import { Text } from "react-native";

export const testID = "amount";
const isTesting = () => global.TEST_MODE === true;

export default () => (
  <>
    <Text>Label</Text>
    <Text
      testID={testID}
      accessible={true}
      accessibilityLabel={isTesting() ? testID : "The price of the item"}
    >
      £50.00
    </Text>
  </>
);
```

That's a load of juggling for every component

Enter `testing-a11y`!

```typescript
import * as React from "react";
import { Text } from "react-native";

import { a11yLabel, a11yOf, a11yProps } from "./lib/testID";

export const amountID = a11yOf("amount", "The price of the item");

export default () => (
  <>
    <Text>Label</Text>
    <Text {...amountID.asProps()}>£50.00</Text>
  </>
);
```

Much simpler! The library takes care of the platform/ a11y switching for you, and allows you to store one reference for testID and a11yLabels in a single place.

Want a unique id for an item in a list? `testing-a11y` can do that too, simply call `amountID(ix)` instead of passing it:

```typescript
import * as React from "react";
import { Text } from "react-native";
import { a11yLabel, a11yOf } from "./lib/testID";

export const amountID = a11yOf("amount", "The price of the item");

export default (props) => (
  <>
    {props.items.map((item, ix) => {
      return (
        <Text>{item.name}</Text>
        <Text {...amountID(ix).asProps()}>{item.amount}</Text>
      )
    })}
  </>
);
```

Imagine you have a common component used all over your app. Each time you use it, because you want it to be easily selectable, you end up passing a differentiating string through to the component to add to the test.

`testing-a11y` simplifies this by allowing you to wrap components. Everything inside the wrapper will have the prefix added to it's testID!

```typescript
import * as React from "react";
import { Button, Text } from "react-native";

import { a11yLabel, a11yOf } from "./lib/testID";

export const submitButtonID = a11yOf("SubmitButton");

export const SubmitButton: React.SFC<{}> = (props) => {
  return (
    <Button
      title={"Submit"}
      onPress={() => void 0}
      {...submitButtonID().asProps()}
    />
  );
};

export default (props) => (
  <>
    <TestIDPrefix value="Form">
      <TestIDPrefix value="InnerForm">
        <SubmitButton />
      </TestIDPrefix>
    </TestIDPrefix>
    <TestIDPrefix value="DifferentForm">
      <SubmitButton />
    </TestIDPrefix>
  </>
);
```

You can now select the two different buttons with:

```typescript
import { a11yProps } from "testing-a11y";

const firstButton = submitButtonID("Form.InnerForm").asTestID();
const otherButton = submitButtonID("DifferentForm").asTestID();
```

Much cleaner!
