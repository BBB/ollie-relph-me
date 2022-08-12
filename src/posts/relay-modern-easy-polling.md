---
title: Relay Modern - Easy Polling
date: 2018-04-27 08:12:02
tags:
  - relay
  - relay-modern
  - react
  - jsx
  - javascript
  - graphql
  - es6
---

[Relay](http://facebook.github.io/relay/en/) & [GraphQl](https://github.com/facebook/graphql) offer the ability to listen for updates via subscriptions. This can be a very powerful tool for creating a real-time collaborative application.

Subscriptions however are not always feasible. It may be that you are working with an API that you don't have control over, or perhaps you simply don't require the fine-grained control that subscriptions require.

This is where polling comes in. Polling will simply re-request your original query after a certain time period. Ensuring eventual syncing with your API.

The [Relay](http://facebook.github.io/relay/en/) docs currently make no mention of it's ability to do polling, so I thought that I'd provide a simple guide here.

> This guide is written for relay-modern@1.5.0

Follow the [Quick Start Guide](http://facebook.github.io/relay/docs/en/quick-start-guide.html) for relay-modern, but instead of returning a promise in your fetch, you should return a `RelayObservable`.

```javascript
import {
  Environment,
  Network,
  Observable,
  RecordSource,
  Store,
} from "relay-runtime";

function fetchQuery(operation, variables) {
  return Observable.create((sink) => {
    fetch("/graphql", {
      method: "POST",
      headers: {
        "Content-Type": "application/json",
      },
      body: JSON.stringify({
        query: operation.text,
        variables,
      }),
    })
      .then((response) => response.json())
      .then((data) => {
        if (data.errors) {
          sink.error(data.errors);
          return;
        }
        sink.next(data);
        sink.complete();
      });
  });
}

const environment = new Environment({
  network: Network.create(fetchQuery),
  store: new Store(new RecordSource()),
});

export default environment;
```

Then using a `<QueryRenderer />` you are able to configure the polling interval that this `Observable` will be called with. The data will automatically flow into your child components.

```jsx
import React from "React";
import { QueryRenderer, graphql } from "react-relay";

import environment from "./environment";

const AllSpecies = (props) => (
  <QueryRenderer
    environment={environment}
    variables={{}}
    query={graphql`
      query AllSpeciesQuery {
        allSpecies {
          edges {
            node {
              classification
              name
              skinColors
            }
          }
        }
      }
    `}
    cacheConfig={{
      poll: 5000,
    }}
    render={(readyState) => {
      if (!readyState.props) {
        return <div>Loading</div>;
      }
      return (
        <div>
          {readyState.props.allSpecies.map((edge) => (
            <div>{edge.node.name}</div>
          ))}
        </div>
      );
    }}
  />
);

export default AllSpecies;
```
