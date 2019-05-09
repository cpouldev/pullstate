---
id: async-server-rendering
title: Resolving async state while server-rendering
sidebar_label: Resolve async state on the server
---

## Universal fetching

Any action that is making use of `useBeckon()` ([discussed in detail here](beckoning-async-action.md)) in the current render tree, like in our example, can have its state resolved on the server before rendering to the client. This allows us to generate dynamic pages on the fly!

**But very importantly:** The asynchronous action code needs to be able to resolve on both the server and client - so make sure that your data-fetching functions are "isomorphic" or "universal" in nature. Examples of such functionality are the [Apollo Client](https://www.apollographql.com/docs/react/api/apollo-client.html) or [Wildcard API](https://github.com/brillout/wildcard-api).

In our example here, this API code would have to be "universally" resolvable:

```tsx
const result = await PictureApi.searchWithTag(tag);
```

## Resolving async state on the server

Until there is a better way to crawl through your react tree, the current way to resolve async state on the server-side while rendering your React app is to simply render it multiple times. This allows Pullstate to register which async actions are required to resolve before we do our final render for the client.

Using the `instance` which we create from our `PullstateCore` object of all our stores:

```tsx
  const instance = PullstateCore.instantiate({ ssr: true });
  
  // (1)
  const app = (
    <PullstateProvider instance={instance}>
      <App />
    </PullstateProvider>
  )

  let reactHtml = ReactDOMServer.renderToString(app);

  // (2)
  while (instance.hasAsyncStateToResolve()) {
    await instance.resolveAsyncState();
    reactHtml = ReactDOMServer.renderToString(app);
  }

  // (3)
  const snapshot = instance.getPullstateSnapshot();

  const body = `
  <script>window.__PULLSTATE__ = '${JSON.stringify(snapshot)}'</script>
  ${reactHtml}`;
```

As marked with numbers in the code:

1. Place your app into a variable for ease of use. After which, we do our initial rendering as usual - this will register the initial async actions which need to be resolved onto our Pullstate `instance`.

2. We enter into a `while()` loop using `instance.hasAsyncStateToResolve()`, which will return `true` unless there is no async state in our React tree to resolve. Inside this loop we immediately resolve all async state with `instance.resolveAsyncState()` before rendering again. This renders our React tree until all state is deeply resolved.

3. Once there is no more async state to resolve, we can pull out the snapshot of our Pullstate instance - and we stuff that into our HTML to be hydrated on the client.

## Selectively resolving async state on the server

If you wish to have the regular behaviour of `useBeckon()` but you don't actually want the server to resolve this asynchronous state (you're happy for it to load on the client-side only). You can pass in an option to `useBeckon()`:

```tsx
const [finished, result, updating] = GetUserAction.useBeckon({ userId }, { ssr: false });
```

Passing in `ssr: false` will cause this action to be ignored in the server asynchronous state resolve cycle.