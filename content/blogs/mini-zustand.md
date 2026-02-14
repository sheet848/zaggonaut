# Build your own mini Zustand / Jotai

Before reaching for Zustand, Jotai, Redux, or signals - it’s worth seeing the core idea in plain JavaScript.

If you understand this loop, every state library suddenly feels familiar.

## The Problem with Libraries

Most developers default to popular state management solutions without asking: What am I actually doing?

You're probably:

- Storing some data
- Letting components know when it changes
- Updating that data
- Notifying listeners of the change

That's it. That's the entire problem.

Most state libraries formalize this concept 

```
store → subscribers → update → notify
```

## The Core Loop

Let me break down what's actually happening:

**Store** : A single object holding your state. Nothing fancy.

**Subscribers** : A list of functions that want to know when state changes.

**Update** : When you change state, you call an update function.

**Notify** : After updating, the store tells all subscribers "hey, I changed."

That's the **entire foundation** of every modern state library.

## Implementation

``` jsx
// minimal reactive store (Zustand / Jotai mental model)

function createStore(initialState) {
  let state = initialState;
  const subscribers = new Set();

  return {
    getState: () => state,
    subscribe: (callback) => {
      subscribers.add(callback);
      return () => subscribers.delete(callback);
    },
    setState: (updater) => {
      state = typeof updater === 'function' ? updater(state) : updater;
      subscribers.forEach(callback => callback(state));
    }
  };
}
```

# How To Use

``` js
// Create a store
const counterStore = createStore({ count: 0 });

// Subscribe to changes
const unsubscribe = counterStore.subscribe((state) => {
  console.log('State changed:', state);
});

// Update state
counterStore.setState({ count: 1 });
// Logs: "State changed: { count: 1 }"

// Functional update
counterStore.setState(state => ({ count: state.count + 1 }));
// Logs: "State changed: { count: 2 }"

// Unsubscribe
unsubscribe();

// Update won't notify anymore
counterStore.setState({ count: 3 });
// Nothing logs
```

This tiny implementation teaches you the core insight that every state library is built on.

Once you understand the store → subscribers → update → notify loop, every library becomes obvious. They're all solving the same problem with different APIs and features on top.

## Integrating with React

``` jsx
function useStore(store) {
  const [state, setState] = React.useState(store.getState());

  React.useEffect(() => {
    return store.subscribe(setState);
  }, [store]);

  return state;
}

// Usage
function Counter() {
  const state = useStore(counterStore);
  return <button onClick={() => counterStore.setState(s => ({ count: s.count + 1 }))}>
    Count: {state.count}
  </button>;
}
```

Done. You've just built a Zustand-like hook system from scratch.

## Advantages

**Lightweight** — No dependency bloat. The entire store is one file.

**Fast** — No middleware overhead, no connect() wrappers, no selectors. Direct updates notify directly.

**Debuggable** — Since it's your code, you understand every line. Debugging is trivial.

**Scalable for Small Apps** — For prototypes, side projects, or parts of apps that don't need Redux's power, this is perfect.

**Educational** — Every time I write this, I learn something. It forces you to understand what state management actually does.

**Flexible** — Want to add persistence? One line to save to localStorage. Want middleware? Add it easily. Want dev tools? Build what you need.

Most of the time, you don't need Redux's time-travel debugging or Zustand's middleware ecosystem. You need something that:

- Holds state
- Notifies when it changes
- Works without framework magic

This delivers exactly that.

## Taking it Further

Once you have this foundation, you can extend it:

**Add Middleware**

javascript

```javascript
setState: (updater) => {
  const prevState = state;
  state = typeof updater === 'function' ? updater(state) : updater;
  middleware.forEach(fn => fn(prevState, state)); // Log, persist, etc.
  subscribers.forEach(callback => callback(state));
}
```

**Add Selectors**

javascript

```javascript
const unsubscribe = store.subscribe(
  (state) => state.count, // selector
  (count) => console.log('Count changed:', count)
);
```

**Add Immer for Immutable Updates**

javascript

```javascript
setState: (updater) => {
  state = produce(state, draft => {
    typeof updater === 'function' ? updater(draft) : Object.assign(draft, updater);
  });
  subscribers.forEach(callback => callback(state));
}
```

The point is: you control everything. You add only what you need.

## Conclusion

State management isn't complicated. The loop is simple:

Store → Subscribers → Update → Notify

Everything else is optimization, ergonomics, and features built on top of this foundation.

Master this pattern, and you've mastered the core of modern state management. Every library suddenly feels familiar. Every design decision makes sense.
