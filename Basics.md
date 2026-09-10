# React Concepts 

## 1. JSX

### What is JSX?

-   JSX = **JavaScript XML**.
-   It lets us write HTML-like syntax inside JavaScript.
-   React uses JSX to describe what the UI should look like.

``` jsx
const element = <h1>Hello World</h1>;
```

### Important points

-   JSX is **not HTML**.
-   JSX gets transformed into JavaScript.
-   You must return **one parent element** unless using a Fragment.
-   JavaScript expressions go inside `{}`.

``` jsx
const name = "John";

return <h1>Hello {name}</h1>;
```

### JSX vs HTML

``` jsx
// JSX
<div className="container">
  <input onChange={handleChange} />
</div>
```

Instead of:

``` html
<div class="container">
  <input onchange="..." />
</div>
```

Important differences: - `class` → `className` - `for` → `htmlFor` -
Event handlers use camelCase: `onClick`, `onChange` - JSX attributes
generally use `{}` for JavaScript values.

### Interview point

> JSX is syntactic sugar that allows us to describe React elements using
> an HTML-like syntax. It is transformed into JavaScript during the
> build process.

------------------------------------------------------------------------

## 2. Components

A component is a **reusable piece of UI**.

``` jsx
function Button() {
  return <button>Click Me</button>;
}
```

Use it:

``` jsx
<Button />
```

### Components should ideally be

-   Small
-   Reusable
-   Focused on one responsibility
-   Predictable
-   Composable

### Function components

Modern React primarily uses function components:

``` jsx
function UserCard() {
  return <div>User</div>;
}
```

### Component tree

For example:

``` text
App
 ├── Header
 ├── Sidebar
 └── UserList
      ├── UserCard
      ├── UserCard
      └── UserCard
```

This is called the **component tree**.

### Senior interview point

A component should generally have a clear responsibility and communicate
with other components through well-defined inputs and outputs.

------------------------------------------------------------------------

## 3. Props

Props are **inputs passed from a parent component to a child**.

``` jsx
function User({ name }) {
  return <h2>{name}</h2>;
}

function App() {
  return <User name="John" />;
}
```

Here:

``` text
App
 ↓
name="John"
 ↓
User
```

### Important properties of props

-   Props are **read-only**.
-   Parent → child data flow.
-   Child should not directly modify props.
-   Props can contain:
    -   Strings
    -   Numbers
    -   Objects
    -   Arrays
    -   Functions
    -   JSX

Example:

``` jsx
<User
  name="John"
  age={30}
  onDelete={handleDelete}
/>
```

### Passing objects

``` jsx
<User user={user} />
```

### Passing functions

``` jsx
<Button onClick={handleClick} />
```

### Interview question

**Can a child modify props?**

No.

Instead, the child can call a function passed by the parent:

``` jsx
function Parent() {
  const handleChange = () => {
    // update parent state
  };

  return <Child onChange={handleChange} />;
}
```

This is commonly called **lifting state up** when the function is used
to update shared state.


## 4. State

State is **data managed by a component that can change over time**.

``` jsx
const [count, setCount] = useState(0);
```

Here:

``` text
count     → current value
setCount  → function to update it
```

Example:

``` jsx
function Counter() {
  const [count, setCount] = useState(0);

  return (
    <button onClick={() => setCount(count + 1)}>
      {count}
    </button>
  );
}
```

### Props vs State

  Props                    State
  ------------------------ -------------------------------
  Passed by parent         Managed by component
  Read-only                Updated using setter/dispatch
  Used for communication   Used for changing data
  Parent → child           Internal/shared state

### Important

Don't mutate state directly.

❌

``` jsx
count = count + 1;
```

❌

``` jsx
user.name = "John";
```

Instead:

``` jsx
setCount(count + 1);
```

or:

``` jsx
setUser({
  ...user,
  name: "John"
});
```


## 5. Component Composition

### What is composition?

Composition means **building larger components by combining smaller
components**.

Instead of creating one huge component:

``` text
Dashboard
 └── Everything
```

Create:

``` text
Dashboard
 ├── Header
 ├── Sidebar
 ├── UserStats
 └── RecentOrders
```

### Example

``` jsx
function Card({ children }) {
  return (
    <div className="card">
      {children}
    </div>
  );
}
```

Usage:

``` jsx
<Card>
  <h2>Profile</h2>
  <p>John Doe</p>
</Card>
```

### Why composition is useful

-   Reusability
-   Maintainability
-   Separation of concerns
-   Flexible UI design
-   Avoids deeply nested conditional logic

### Senior interview point

> React favors composition over inheritance for reusing UI behavior and
> structure.


## 6. Conditional Rendering

Conditional rendering means **rendering different UI based on a
condition**.

### `if`

``` jsx
function User({ isLoggedIn }) {
  if (isLoggedIn) {
    return <Dashboard />;
  }

  return <Login />;
}
```

### Ternary

``` jsx
{isLoggedIn ? <Dashboard /> : <Login />}
```

Use when there are two possibilities.

### `&&`

``` jsx
{isAdmin && <AdminPanel />}
```

Meaning:

> If `isAdmin` is true, render `AdminPanel`.

### Important pitfall

Be careful with:

``` jsx
{count && <Message />}
```

If `count = 0`, React can render `0`.

Better:

``` jsx
{count > 0 && <Message />}
```

### Multiple conditions

Avoid complicated nested ternaries.

Prefer: - Early returns - Helper functions - Separate components

------------------------------------------------------------------------

## 7. Lists

React commonly renders arrays using `.map()`.

``` jsx
const users = ["John", "Jane", "Mike"];

return (
  <ul>
    {users.map(user => (
      <li>{user}</li>
    ))}
  </ul>
);
```

Usually you need a `key`:

``` jsx
{users.map(user => (
  <li key={user}>{user}</li>
))}
```

### Why use `.map()`?

It transforms:

``` text
Array of data
     ↓
Array of React elements
```


## 8. Keys

### What is a key?

A key is a **stable identifier for an item in a list**.

``` jsx
users.map(user => (
  <UserCard key={user.id} user={user} />
))
```

### Why does React need keys?

React uses keys to understand: - Which item was added? - Which item was
removed? - Which item changed? - Which item moved?

This helps React efficiently reconcile the UI.

### Good key

``` jsx
key={user.id}
```

### Bad key

``` jsx
key={Math.random()}
```

Because it changes every render.

### Index as key

``` jsx
key={index}
```

This can be problematic when the list can: - Reorder - Insert items -
Delete items

Example:

``` text
Before:
A
B
C

After inserting X:
X
A
B
C
```

With indexes, React may associate the wrong component state with items.

### Senior interview point

> Keys should be stable, unique among siblings, and derived from the
> identity of the data rather than its position.


## 9. Children

`children` is a special prop containing whatever is placed between a
component's opening and closing tags.

``` jsx
<Card>
  <h2>Hello</h2>
  <p>Welcome</p>
</Card>
```

Inside `Card`:

``` jsx
function Card({ children }) {
  return (
    <div className="card">
      {children}
    </div>
  );
}
```

Think:

``` text
<Card>
   ↓
children
   ↓
<h2>Hello</h2>
<p>Welcome</p>
```

### Why is children useful?

It makes components flexible.

For example:

``` jsx
<Modal>
  <LoginForm />
</Modal>
```

or:

``` jsx
<Modal>
  <DeleteConfirmation />
</Modal>
```

Same `Modal`, different content.


## 10. Fragments

A component often needs to return multiple elements.

This won't work:

``` jsx
return (
  <h1>Hello</h1>
  <p>Welcome</p>
);
```

You need a parent.

### Normal wrapper

``` jsx
return (
  <div>
    <h1>Hello</h1>
    <p>Welcome</p>
  </div>
);
```

But this adds an unnecessary `<div>` to the DOM.

### Fragment

``` jsx
return (
  <>
    <h1>Hello</h1>
    <p>Welcome</p>
  </>
);
```

This groups elements **without adding an extra DOM element**.

### Explicit Fragment

``` jsx
import { Fragment } from "react";

return (
  <Fragment>
    <h1>Hello</h1>
    <p>Welcome</p>
  </Fragment>
);
```

Explicit fragments are useful when you need a `key`.

``` jsx
items.map(item => (
  <Fragment key={item.id}>
    <dt>{item.name}</dt>
    <dd>{item.description}</dd>
  </Fragment>
))
```


## 11. Event Handling

React handles events using props such as:

``` jsx
onClick
onChange
onSubmit
onMouseEnter
```

Example:

``` jsx
function Button() {
  const handleClick = () => {
    console.log("Clicked");
  };

  return (
    <button onClick={handleClick}>
      Click
    </button>
  );
}
```

### Important mistake

❌ Don't call the function during render:

``` jsx
<button onClick={handleClick()}>
```

This executes immediately.

✅ Pass the function:

``` jsx
<button onClick={handleClick}>
```

### Passing arguments

Use an arrow function:

``` jsx
<button onClick={() => handleDelete(id)}>
  Delete
</button>
```

### Event object

React gives your handler an event object:

``` jsx
function handleChange(event) {
  console.log(event.target.value);
}
```

Example:

``` jsx
<input onChange={handleChange} />
```

### Form submission

``` jsx
function handleSubmit(event) {
  event.preventDefault();

  // submit form
}
```

``` jsx
<form onSubmit={handleSubmit}>
  ...
</form>
```


