# React `useEffect` — Deep Dive

## 1. Stale Closures

A **closure** is a function that remembers variables from the render where it was created.

A **stale closure** happens when a callback continues using an old state or prop value.

```jsx
useEffect(() => {
  const id = setInterval(() => {
    console.log(count);
  }, 1000);

  return () => clearInterval(id);
}, []);

// Output : 0 0 0 0 ....
```

If `count` changes but the Effect has `[]`, the interval callback can keep seeing the initial `count`.

### Solutions

Add the dependency:

```jsx
useEffect(() => {
  const id = setInterval(() => {
    console.log(count);
  }, 1000);

  return () => clearInterval(id);
}, [count]);
```

Or use a functional state update when updating state:

```jsx
useEffect(() => {
  const id = setInterval(() => {
    setCount(c => c + 1);
  }, 1000);

  return () => clearInterval(id);
}, []);
```

### Interview answer

> **A stale closure occurs when a callback created during an earlier render continues to reference old props or state.**


## 2. Exhaustive Dependencies

The `react-hooks/exhaustive-deps` lint rule helps identify missing dependencies.

### Missing dependency

```jsx
useEffect(() => {
  console.log(userId);
}, []);
```

### Correct

```jsx
useEffect(() => {
  console.log(userId);
}, [userId]);
```

### Why?

An Effect should stay synchronized with the reactive values it uses.

```text
Effect uses userId
       ↓
userId changes
       ↓
Effect should potentially synchronize again
```

Avoid simply disabling the lint rule. Instead ask:

> **Why does this Effect depend on this value, and can I restructure the code so the dependencies are correct?**



## 3. Infinite Loops

An Effect can accidentally cause an infinite render loop.

```jsx
const [count, setCount] = useState(0);

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
...
```

#### Another common cause: unstable object dependencies

```jsx
const options = {
  page: 1,
};

useEffect(() => {
  fetchData(options);
}, [options]);
```

If `options` is recreated on every render, its reference changes.

```jsx
{} === {} // false
```

#### Debug checklist

1. Does the Effect update state?
2. Is that state in the dependency array?
3. Does the Effect create a new object/function/array every render?
4. Is a dependency unnecessarily unstable?
5. Does the Effect actually need to exist?


## 4. Race Conditions

Race conditions commonly happen with asynchronous requests.

Example:

```text
User selects "React"
       ↓
Request A starts

User selects "Vue"
       ↓
Request B starts
```

If B finishes first and A finishes later:

```text
B → Vue data displayed
A → React data displayed
```

The UI may incorrectly show stale data.

### Solution: AbortController

```jsx
useEffect(() => {
  const controller = new AbortController();

  async function loadUser() {
    try {
      const response = await fetch(
        `/api/users/${userId}`,
        { signal: controller.signal }
      );

      const data = await response.json();
      setUser(data);
    } catch (error) {
      if (error.name !== "AbortError") {
        console.error(error);
      }
    }
  }

  loadUser();

  return () => {
    controller.abort();
  };
}, [userId]);
```

Flow:

```text
userId = 1
   ↓
Request A

userId = 2
   ↓
Cleanup
   ↓
Abort Request A
   ↓
Request B
```


## 5. Effect vs Event Handler

This is a **very important senior interview distinction**.

### Event Handler

Use an event handler when something happens because of a **specific user action**.

```jsx
function handleSubmit() {
  saveData();
}
```

```jsx
<button onClick={handleSubmit}>
  Save
</button>
```

### Effect

Use an Effect when the component needs to **synchronize with an external system based on rendered state/props**.

```jsx
useEffect(() => {
  document.title = `Count: ${count}`;
}, [count]);
```

### Comparison

| Event Handler | Effect |
|---|---|
| User interaction | Render/state synchronization |
| `onClick` | `useEffect` |
| `onSubmit` | `useEffect` |
| `onChange` | `useEffect` |
| User intentionally performs an action | External system needs synchronization |
| Caused by an event | Caused by reactive changes |

### Example

Often unnecessary:

```jsx
useEffect(() => {
  if (isSubmitted) {
    sendAnalytics();
  }
}, [isSubmitted]);
```

Better when submission itself is the trigger:

```jsx
function handleSubmit() {
  saveData();
  sendAnalytics();
}
```

### Interview answer

> **Event handlers respond to user interactions; Effects synchronize with external systems as a consequence of rendering or reactive state changes.**


## 6. Fetch Patterns

### Pattern 1: Basic Fetch

```jsx
useEffect(() => {
  async function fetchUsers() {
    const response = await fetch("/api/users");
    const data = await response.json();

    setUsers(data);
  }

  fetchUsers();
}, []);
```

Generally don't make the Effect callback itself `async`:

```jsx
// Avoid
useEffect(async () => {
  ...
}, []);
```

- Instead define an async function inside the Effect.
- useEffect should return cleanup or undefined


## 7. Fetch Based on a Dependency

```jsx
function User({ userId }) {
  const [user, setUser] = useState(null);

  useEffect(() => {
    async function fetchUser() {
      const response = await fetch(`/api/users/${userId}`);
      const data = await response.json();

      setUser(data);
    }

    fetchUser();
  }, [userId]);

  return <div>{user?.name}</div>;
}
```

Flow:

```text
userId = 1
   ↓
Fetch user 1

userId = 2
   ↓
Fetch user 2
```


## 8. Fetch + Loading + Error

A real application often needs loading and error states.

```jsx
const [data, setData] = useState(null);
const [loading, setLoading] = useState(true);
const [error, setError] = useState(null);

useEffect(() => {
  let ignore = false;

  async function fetchData() {
    setLoading(true);
    setError(null);

    try {
      const response = await fetch("/api/users");

      if (!response.ok) {
        throw new Error("Request failed");
      }

      const result = await response.json();

      if (!ignore) {
        setData(result);
      }
    } catch (error) {
      if (!ignore) {
        setError(error);
      }
    } finally {
      if (!ignore) {
        setLoading(false);
      }
    }
  }

  fetchData();

  return () => {
    ignore = true;
  };
}, []);
```

The `ignore` pattern prevents stale requests from updating the component after the Effect has been superseded.

For actual network cancellation, `AbortController` is often preferable when supported.



## 9. Modern Data Fetching

Manually doing this is not always the best architecture:

```jsx
useEffect(() => {
  fetch(...);
}, []);
```

Modern applications may use:

- React framework data-loading mechanisms
- TanStack Query
- SWR
- Server Components / server-side data fetching where applicable
- Application-specific data-fetching abstractions

These can provide:

```text
Caching
↓
Deduplication
↓
Retries
↓
Loading states
↓
Error handling
↓
Refetching
↓
Request synchronization
```

### Senior interview answer

> **`useEffect` can perform client-side data fetching, but for complex applications I'd generally prefer the framework's data-loading mechanism or a dedicated data-fetching library when appropriate.**


## 10. Stale Closure + Fetch

An async callback can use old values if dependencies are incorrect.

### Correct

```jsx
useEffect(() => {
  async function fetchUser() {
    const response = await fetch(`/api/users/${userId}`);
    const data = await response.json();

    setUser(data);
  }

  fetchUser();
}, [userId]);
```

### Problematic

```jsx
useEffect(() => {
  async function fetchUser() {
    // ...
  }

  fetchUser();
}, []);
```

If `userId` changes, the Effect may continue using the initial `userId`.



## 11. Common Mistakes to Avoid

### Missing dependencies


```jsx
useEffect(() => {
  console.log(userId);
}, []);
```


```jsx
useEffect(() => {
  console.log(userId);
}, [userId]);
```

### Effect for derived state


```jsx
useEffect(() => {
  setFullName(firstName + lastName);
}, [firstName, lastName]);
```


```jsx
const fullName = firstName + lastName;
```

### Effect for user action


```jsx
useEffect(() => {
  if (clicked) {
    submitForm();
  }
}, [clicked]);
```


```jsx
function handleClick() {
  submitForm();
}
```

### Unstable dependency

Be careful with:

```jsx
const options = { page: 1 };

useEffect(() => {
  fetchData(options);
}, [options]);
```

`options` can have a new identity on each render.

### Updating your own dependency


```jsx
useEffect(() => {
  setCount(count + 1);
}, [count]);
```

This can create an infinite loop.

---

## 12. Quick Mental Model

When you see an Effect, ask these questions:

```text
1. What external system am I synchronizing with?
                    ↓
2. What values does the Effect use?
                    ↓
3. Are those values dependencies?
                    ↓
4. What happens when the dependency changes?
                    ↓
5. What needs to be cleaned up?
```

For async work, add:

```text
6. What happens if an old request finishes after a new request?
```

---

## 13. Senior Interview Cheat Sheet

| Topic | Simple explanation |
|---|---|
| **Stale closure** | Callback sees values from an older render |
| **Exhaustive dependencies** | Include reactive values used by the Effect |
| **Infinite loop** | Effect updates something that causes itself to run again |
| **Race condition** | Older async result can overwrite newer data |
| **AbortController** | Cancels an in-flight fetch |
| **Effect** | Synchronizes with external systems |
| **Event handler** | Responds to user interaction |
| **Derived state** | Usually calculate during render, not with Effect |
| **Fetch in Effect** | Valid for client-side fetching, but not always ideal |
| **Data-fetching library** | Handles caching, deduplication, retries, refetching, etc. |
| **Cleanup** | Stops/removes/cancels previous synchronization |

---



