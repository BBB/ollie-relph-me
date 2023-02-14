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

In a TypeScript based project, it is not typesafe, and does not alert you to incompatibilities in your test code. It is actively dangerous and helps cover up issues in your codebase.

The paths passed to `jest.mock` are not refactorable, nor easy to maintain.

***

There is another way! Describe your dependencies as `interface`s write "real" implementations alongside fake ones that provide the functionality required by your test. Pass these dependencies as defaulted parameters to your code.

    
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
