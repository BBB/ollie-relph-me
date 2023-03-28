---
date: 2023-02-14
title: Using jest.mock is the opposite of TDD
tags:
- Mocking
- Testing
- Jest
- TDD
image: ''

---
Using `jest.mock` is an after-thought. It is the opposite of a test-driven mentality. It says that the code is so untestable that you need to use a HACK based on a transformation to test your code.

In a TypeScript based project, it is not type-safe, and does not alert you to incompatibilities in your test code. It is actively dangerous and helps cover up issues in your codebase.

The paths passed to `jest.mock` are not simple to refactor, nor easy to maintain.

This is also true for monkey-patching global objects. (think `fetch`, `window.location`, etc.)

There is another way! 

### 1) Describe your dependencies as interfaces 

```typescript
type Widget = { id: string };

type HttpClient = (
  input: RequestInfo | URL,
  init?: RequestInit
) => Promise<Response>;

interface WidgetClient {
  getWidget(id: string): Promise<Widget>;
  saveWidget(widget: Widget): Promise<void>;
}
```

### 2) Write "Real" implementation

Note how this takes a `HttpClient`. We should also unit test this ApiClient using a fake `HttpClient` implementation.

```typescript
    
class WidgetApiClient implements WidgetClient {
  constructor(private httpClient: HttpClient = window.fetch) {}
  getWidget(id: string) {
    return this.httpClient("/url/" + id)
      .then((res) => res.json())
      .then(({ data }) => data as Widget);
  }
  saveWidget(widget: Widget) {
    return this.httpClient("/url" + widget.id, {
      method: "PUT",
      body: JSON.stringify(widget),
    }).then(() => undefined);
  }
}
```

### 3) Write "Fake" implementation that provides the functionality required by your test

```typescript
    
class WidgetFakeClient implements WidgetClient {
  constructor(private widgetStore: Record<string, Widget> = {}) {}
  getWidget(id: string) {
    return Promise.resolve(this.widgetStore[id]!);
  }
  saveWidget(widget: Widget) {
    this.widgetStore[widget.id] = widget;
    return Promise.resolve();
  }
}
```
### 3) Pass these dependencies as defaulted parameters to your code, that can then be overridden in tests

```typescript
    
const Widgets = (props = { widgetClient = new WidgetApiClient() }) => {
  ...
}

```

Now you've got a fake that you can use alongside your test, or in cases where the api is unstable.