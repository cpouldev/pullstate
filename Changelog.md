### 1.10.0

Updated `immer` to `^5.0.0` which has better support for `Map` and `Set`.

Updated `fast-deep-equal` to use `^3.0.0`, which has support for `Map` and `Set` types, allowing you to use these without worry in your stores.

## 1.9.0

* Added the ability to pass a third option to `useStoreState()` - this allows the our listener to be dynamically updated to listen to a different sub-state of our store. Similar to how the last argument in `useEffect()` and such work.
  * see https://github.com/lostpebble/pullstate/issues/22

#### React Suspense!

* You can now use Async Actions with React Suspense. Simply use them with the format: `myAction.read(args)` inside a component which is inside of `<Suspend/>`.

## 1.8.0

* Added the passable option `{ dormant: true }` to Async Function's `useBeckon()` or `useWatch()`, which will basically just make the action completely dormant - no execution or hitting of cache or anything, but will still respect the option `{ holdPrevious: true }`, returning the last completed result for this action if it exists.

### 1.7.3

[TypeScript] Minor type updates for calling `useStore()` directly on one of your stores, so that the "sub-state" function gets the store's state interface correctly.

### 1.7.2

Minor quality of life update, able to now set successful cached payloads directly in the cache using

```
setCachedPayload(args, payload, options)
```

Much thanks again to @bitttttten for the pull request.

### 1.7.1

Slight change to how `Store.update()` runs when accepting an array of updaters. It now runs each update separately on the state, allowing for updates further down the line to act on previous updates (still triggers re-renders of your React components as if it were a single update).

Thanks to @bitttttten for a fix which allows passing no arguments when using `createPullstateCore()`.

## 1.7.0

Allow optional passing of multiple update functions to `Store.update()` in the format of an array.

For example:

```ts
UIStore.update([setDarkMode, setTypography("Roboto)]);
```

This allows "batching" actions which are defined in smaller, modular functions together in one go. It makes the pattern of creating "updater" functions for your store a little more easier to work with, as you can combine multiples of them in one update now.

### 1.6.4

- Exported `TUpdateFunction` so that we can more easily create update functions corresponding to pullstate.

### 1.6.3

- Added convenience method `useInstance` or `PullstateCore.useInstance` (for better typing on your stores) - which gives direct access to your Pullstate instance which was passed in to `PullstateProvider`.

### 1.6.2

- Added export for `PullstateContext` for more customized usage of Pullstate.

### 1.6.1

- **[async][bugfix]** fixed problem with multiple beckoned actions infinite looping for same arguments
- Allow for passing a non-object as an argument to an async action (`string` / `boolean` etc.)

## 1.6.0

Added the ability to hold onto previously resolved action results (if they were successful) until the new action resolves, when using a `useWatch()` or `useBeckon()`:

- Pass `holdPrevious: true` as an option to either `useWatch()` or `useBeckon()` to enable this.
- When a new action is running on top of an old result, the returned value from your action hooks will now have `started = true`, `finished = true`, `result = sameResult` and a final value to check called `updating = true`:
  - `[true, true, result, true]` (for `useWatch()`)
  - `[true, result, true]` for (`useBeckon()`)

### 1.5.1

Added `--strictNullChecks` in TypeScript and fixed loads of types which had undefined / null options. Should let Pullstate play nicely with the other children now.

## 1.5.0

Allow selecting a subset of passed arguments too an async function to create the fingerprint. This is purely for performance reasons when you want to pass in large data sets.

Pass an extra option when creating the Async Action: `subsetKey: (args) => subset` - basically it takes the arguments given, and allows you to return subset of those arguments which pullstate will use internally to create cache fingerprints.

### 1.4.1

- Added `immer` as direct dependency. Was `peerDependency` before - but this is not sufficient when requiring certain versions of `immer` for new functionality. Also `peerDependency` gives errors to users whose projects don't use `immer` outside of `pullstate`.

## 1.4.0

- Added the ability to listen for change patches on an entire store, using `Store.listenToPatches(patchListener)`.

- Fixed a bug where applying patches to stores didn't trigger the new optimized updates.
- Fixed bug with Reactions running twice

### 1.3.1

- Fixed Reactions to work with path change optimizations (see `1.2.0`). Previously only `update()` kept track of path changes - forgot to add path tracking to Reactions.

## 1.3.0

- Expanded on `getCached()`, `setCached()` and `updateCached()` on Async Actions - and made sure they can optionally notify any listeners on their cached values to re-render on changes.
- Added `clearAllUnwatchedCache()` on Async Actions for quick and easy garbage collection.
- Added `timeCached` as a passed argument to the `cacheBreakHook()`, allowing for easier cache invalidation against the time the value was last cached.

## 1.2.0

New experimental optimized updates (uses immer patches internally). To use, your state selections need to be made using paths - and make use of the new methods and components `useStoreStateOpt` and `<InjectStoreStateOpt>` respectively.

Instead of passing a function, you now pass an array of path selections. The state returned will be an array of values per each state selection path. E.g:

```ts
const [isDarkMode] = useStoreStateOpt(UIStore, [["isDarkMode"]]);
```

The performance benefits stem from Pullstate not having to run equality checks on the results of your selected state and then re-render your component accordingly, but instead looks at the immer update patches directly for which paths changed in your state and re-renders the listeners on those paths.

## 1.1.0

Fixed issue with postActionHook not being called on the server for Async Actions.

Added the following methods on Async Actions:

- `setCached()`
- `updateCached()`

For a more finer-grained control of async action cache.

`updateCached()` functions exactly the same as `update()` on stores, except it only runs on a previously successfully returned cached value. If nothing is cached, nothing is run.

## 1.0.0-beta.7

Replaced `shallowEqual` from `fbjs` with the tiny package `fast-deep-equal` for object comparisons in various parts of the lib.

## 1.0.0-beta.6

Fixed the `postActionHook` to work correctly when hitting a cached value.

## 0.8.0.alpha-2

Added `IPullstateInstanceConsumable` as an export to help people who want to create code using the Pullstate stores' instance.

## 0.8.0.alpha-1

Some refactoring of the Async Actions and adding of hooks for much finer grained control:

`shortCicuitHook()`: Run checks to resolve the action with a response before it even sets out.

`breakCacheHook()`: When an action's state is being returned from the cache, this hook allows you to run checks on the current cache and your stores to decide whether this action should be run again (essentially flushing / breaking the cache).

`postActionHook()`: This hook allows you to run some things after the action has resolved, and most importantly allows code to run after each time we hit the cached result of this action as well. This is very useful for interface changes which need to change / update outside of the action code.

`postActionHook()` is run with a context variable which tells you in which context it was run, one of: CACHE, SHORT_CIRCUIT, DIRECT_RUN

These hooks should hopefully allow even more boilerplate code to be eliminated while working in asynchronous state scenarios.

## 0.7.1

- Made the `isResolved()` function safe from causing infinite loops (Async Action resolves, but the state of the store still makes `isResolved()` return false which causes a re-trigger when re-rendering - most likely happens when not checking for error states in `isResolved()`) - instead posting an error message to the console informing about the loop which needs to be fixed.

## 0.7.0

**:warning: Replaced with async action hooks above in 0.8.0**

Added the options of setting an `isResolve()` synchronous checking function on Async Actions. This allows for early escape hatching (we don't need to run this async action based on the current state) and cache busting (even though we ran this Async Action before and we have a cached result, the current state indicates we need to run it again).

You can set it like so:

```typescript jsx
const loadEntity = PullstateCore.createAsyncAction<{ id: string }>(
  async ({ id }, { EntityStore }) => {
    const resp = await endpoints.getEntity({ id });

    if (resp.positive) {
      EntityStore.update(s => {
        s.viewingEntity = resp.payload;
      });
      return successResult();
    }

    return errorResult(resp.endTags, resp.endMessage);
  },

  // This second argument is the isResolved() function

  ({ id }, { EntityStore }) => {
    const { viewingEntity } = EntityStore.getRawState();

    if (viewingEntity !== null && viewingEntity.id === id) {
      return successResult();
    }

    return false;
  }
);
```

It has the same form as the regular Async Action function, injecting the arguments and the stores - but needs to return a synchronous result of either `false` or the expected end result (as if this function would have run asynchronously).

## 0.6.0

- Added "reactions" to store state. Usable like so:

```typescript jsx
UIStore.createReaction(
  s => s.valueToListenForChanges,
  (draft, original, watched) => {
    // do something here when s.valueToListenForChanges changes
    // alter draft as usual - like regular update()
    // watched = the value returned from the first function (the selector for what to watch)
  }
);
```
