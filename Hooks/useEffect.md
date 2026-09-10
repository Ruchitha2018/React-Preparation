# React `useEffect`

## 1. Purpose

`useEffect` is used to **synchronize a React component with something outside React**.

### Common external systems

- API requests
- WebSocket connections
- Browser event listeners
- Timers
- DOM APIs
- Third-party libraries
- Subscriptions

### Basic syntax

```jsx
useEffect(() => {
  // synchronize with external system

  return () => {
    // cleanup
  };
}, [dependencies]);
```

### Key interview statement

> **`useEffect` is primarily for synchronizing React with external systems, not for running arbitrary code after every render.**

---

## 2. When Does an Effect Run?

The important flow is:

```text
State / Props change
       ↓
React renders component
       ↓
React commits changes to DOM
       ↓
Browser paints
       ↓
Effect runs
```

For a normal `useEffect`, think:

> **Render → Commit → Browser Paint → Effect**

There are nuances around interaction-driven effects and React scheduling, so avoid saying "`useEffect` always runs after paint" as an absolute rule.

### Safe interview statement

> **`useEffect` runs after React commits the update, and generally after the browser paints.**

---

## 3. Dependency Array

The dependency array tells React:

> **"When should this Effect be synchronized again?"**

### No dependency array

```jsx
useEffect(() => {
  console.log("effect");
});
```

Runs after **every render**.

```text
Render → Effect
Render → Effect
Render → Effect
```

---

### Empty dependency array

```jsx
useEffect(() => {
  console.log("effect");
}, []);
```

Runs for the component's **initial synchronization**.

Conceptually:

```text
Mount
 ↓
Effect
```

### Important

In development with **Strict Mode**, React may run:

```text
Setup → Cleanup → Setup
```

to help detect bugs.

So don't blindly say "`[]` means exactly once."

---

### With dependencies

```jsx
useEffect(() => {
  console.log(userId);
}, [userId]);
```

Runs:

```text
Initial render
      ↓
Effect

userId changes
      ↓
Effect again
```

If `userId` doesn't change:

```text
Re-render
   ↓
No Effect
```

---

## 4. Dependency Rule

Every **reactive value used inside the Effect** generally needs to be included in the dependency array.

Example:

```jsx
function User({ userId }) {
  useEffect(() => {
    fetchUser(userId);
  }, [userId]);
}
```

`userId` is used by the Effect, so it belongs in:

```jsx
[userId]
```

### Interview trap

Don't think:

> "I put something in the dependency array because I want the Effect to run when that value changes."

Think:

> **"The dependency array describes the reactive values that the Effect depends on."**

---

## 5. Effect Lifecycle

Think of an Effect as having **setup and cleanup**.

```jsx
useEffect(() => {
  // SETUP

  return () => {
    // CLEANUP
  };
}, [value]);
```

Suppose `value` changes.

The lifecycle is:

```text
Initial render
     ↓
Effect setup
     ↓
value changes
     ↓
Cleanup previous Effect
     ↓
Effect setup again
```

### Important rule

> **Before React runs a new Effect setup, it runs the cleanup from the previous Effect.**

When the component is removed:

```text
Component unmounts
       ↓
Cleanup
```

---

## 6. Cleanup

Cleanup is used to **undo whatever the Effect setup did**.

### Event listener

```jsx
useEffect(() => {
  function handleResize() {
    console.log(window.innerWidth);
  }

  window.addEventListener("resize", handleResize);

  return () => {
    window.removeEventListener("resize", handleResize);
  };
}, []);
```

Setup:

```text
addEventListener
```

Cleanup:

```text
removeEventListener
```

---

### Timer

```jsx
useEffect(() => {
  const id = setInterval(() => {
    console.log("Hello");
  }, 1000);

  return () => {
    clearInterval(id);
  };
}, []);
```

Setup:

```text
setInterval
```

Cleanup:

```text
clearInterval
```

---

### WebSocket

```jsx
useEffect(() => {
  const socket = connect();

  return () => {
    socket.disconnect();
  };
}, []);
```

Setup:

```text
connect
```

Cleanup:

```text
disconnect
```

### Easy rule

> **If your Effect creates something that needs to be stopped, removed, cancelled, or disconnected, cleanup is probably needed.**

---

## 7. External Synchronization

This is one of the **most important senior-level concepts**.

React manages:

```text
State
Props
UI
```

But sometimes your component needs to communicate with something React doesn't control:

```text
React
  ↓
External System
```

### API

```jsx
useEffect(() => {
  fetch(`/api/users/${userId}`);
}, [userId]);
```

### Browser API

```jsx
useEffect(() => {
  document.title = `User ${userId}`;
}, [userId]);
```

### Event listener

```jsx
useEffect(() => {
  window.addEventListener("resize", handleResize);

  return () => {
    window.removeEventListener("resize", handleResize);
  };
}, []);
```

### Subscription

```jsx
useEffect(() => {
  const unsubscribe = subscribe(handleUpdate);

  return unsubscribe;
}, []);
```

### Mental model

```text
React state/props
       ↓
     Effect
       ↓
External system
```

When dependencies change, React re-synchronizes.


## 8. Common Anti-Patterns

### Anti-pattern 1: Derived State Inside Effect

#### Bad

```jsx
const [fullName, setFullName] = useState("");

useEffect(() => {
  setFullName(firstName + " " + lastName);
}, [firstName, lastName]);
```

This is unnecessary.

### Better

```jsx
const fullName = firstName + " " + lastName;
```

### Why?

Because `fullName` is **derived from existing state/props**.

You don't need an Effect.


## Anti-pattern 2: Using Effect for Event Handling

### Often unnecessary

```jsx
useEffect(() => {
  if (submitted) {
    sendAnalytics();
  }
}, [submitted]);
```

If the action happens because the user clicked Submit, the logic usually belongs in the event handler:

```jsx
function handleSubmit() {
  sendAnalytics();
}
```

### Rule

> **If something happens because of a specific user interaction, an event handler is often the better place.**

Effects are for **synchronization caused by rendering/state changes**, not for replacing event handlers.


## Anti-pattern 3: Infinite Loops

Example:

```jsx
useEffect(() => {
  setCount(count + 1);
}, [count]);
```

Flow:

```text
count changes
   ↓
Effect runs
   ↓
setCount()
   ↓
render
   ↓
count changes
   ↓
Effect runs
   ↓
setCount()
   ↓
...
```

This creates an infinite update loop.


## 9. Fetching Data — Senior Interview Point

You might see:

```jsx
useEffect(() => {
  fetch(`/api/users/${userId}`)
    .then(...);
}, [userId]);
```

This is valid in many applications, but modern React applications often use **data-fetching libraries or framework mechanisms** instead of manually managing fetching with Effects.

### These tools can handle

- Caching
- Deduplication
- Loading states
- Errors
- Race conditions
- Refetching

### Better interview answer

Don't simply say:

> "We use `useEffect` for API calls."

Instead say:

> **"An Effect can synchronize with an external data source, but data fetching doesn't necessarily have to be implemented manually with `useEffect`."**


## Race Conditions

Consider:

```text
userId = 1
   ↓
Request A starts

userId = 2
   ↓
Request B starts
```

Suppose B finishes first:

```text
B → user 2
```

Then A finishes:

```text
A → user 1
```

Now the UI could incorrectly show user 1.

### Solution

Asynchronous Effects need careful handling, often with:

- Request cancellation
- `AbortController`
- Ignoring stale results
- Data-fetching libraries

## Senior-Level Mental Model

### Mental model

```text
             React
               │
        props / state
               │
               ▼
            Render
               │
               ▼
          DOM Commit
               │
               ▼
          Browser Paint
               │
               ▼
            Effect
               │
               ▼
       External System
```

When dependencies change:

```text
Dependency changes
       ↓
React renders
       ↓
Commit
       ↓
Cleanup previous Effect
       ↓
Browser paint
       ↓
Setup new Effect
```

---

