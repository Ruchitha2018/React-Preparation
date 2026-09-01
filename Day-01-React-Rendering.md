# React Rendering

## What is Rendering in React?

In simple terms:

> **Rendering is the process of React determining what the UI should look like based on the current props and state.**

Example:

```jsx
function Counter() {
  const [count, setCount] = useState(0);

  return <h1>{count}</h1>;
}
```

Initially:

```text
count = 0
  ↓
React renders component
  ↓
<h1>0</h1>
  ↓
DOM shows 0
```

When:

```js
setCount(1);
```

React doesn't immediately change the DOM.

Instead:

```text
setCount(1)
   ↓
State update scheduled
   ↓
React renders again
   ↓
New React element: <h1>1</h1>
   ↓
React compares old vs new
   ↓
Commit necessary DOM changes
   ↓
DOM becomes <h1>1</h1>
```

### Important Interview Point

**Rendering ≠ updating the DOM.**

Rendering means:

> "React calculates what the UI should look like."

The DOM update happens later during the **commit phase**.

---

## What Causes a React Render?

A component can render because of several things.

### 1. State update

```jsx
setCount(count + 1);
```

### 2. Parent component renders

```jsx
function Parent() {
  const [count, setCount] = useState(0);

  return <Child />;
}
```

When `Parent` renders, React normally evaluates `Child` again as well.

### 3. Props change

```jsx
<Child name="John" />
```

Changing it to:

```jsx
<Child name="Mike" />
```

can cause the child to render.

### 4. Context value changes

```jsx
const value = useContext(MyContext);
```

If the consumed context value changes, the component can re-render.

### 5. External store update

For example:

```js
useSyncExternalStore();
```

or updates from state-management libraries.

---

## Does Every Render Update the DOM?

**No.**

This is one of the most important React concepts.

Example:

```jsx
function Counter() {
  const [count, setCount] = useState(0);

  return <h1>Hello</h1>;
}
```

Suppose:

```js
setCount(1);
```

React renders again.

But the output is still:

```jsx
<h1>Hello</h1>
```

React realizes:

```text
Previous UI:
<h1>Hello</h1>

New UI:
<h1>Hello</h1>

No meaningful difference
```

So:

```text
Render happens
      ↓
Comparison happens
      ↓
No DOM change required
```

Therefore:

> **A re-render does not necessarily mean a DOM update.**

---

## Render Phase vs Commit Phase

This is a **very important senior-level interview topic**.

React's update process can broadly be divided into:

```text
              React Update
                   │
          ┌────────┴────────┐
          ↓                 ↓
     Render Phase       Commit Phase
          │                 │
     Calculate UI       Update DOM
     Reconciliation    Run layout effects
          │                 │
          └────────┬────────┘
                   ↓
              Browser Paint
                   ↓
             Passive Effects
```

Let's understand each.

---

## Render Phase

The **render phase** determines what the next UI should be.

React:

- Calls components
- Evaluates JSX
- Creates React elements
- Performs reconciliation
- Determines what needs to change

Example:

```jsx
function Counter() {
  const [count, setCount] = useState(0);

  return (
    <div>
      <h1>{count}</h1>
      <button onClick={() => setCount(count + 1)}>
        Increment
      </button>
    </div>
  );
}
```

After:

```js
setCount(1);
```

React executes the component again:

```jsx
Counter()
```

and gets:

```jsx
<div>
  <h1>1</h1>
  <button>Increment</button>
</div>
```

React now has:

```text
Previous result          New result

<h1>0</h1>               <h1>1</h1>
```

It determines:

```text
<h1> text changed
```

---

## Important Property of Render Phase

The render phase should be **pure**.

Meaning:

> Rendering should calculate UI without causing side effects.

Bad:

```jsx
function Component() {
  document.title = "Hello";

  return <h1>Hello</h1>;
}
```

Don't perform side effects during render.

Instead:

```jsx
useEffect(() => {
  document.title = "Hello";
}, []);
```

### Why?

React may perform rendering work more than once, abandon work, or restart work, especially with modern concurrent rendering.

---

## Commit Phase

Once React finishes determining the changes, it enters the **commit phase**.

Here React applies the required changes to the actual DOM.

Example:

```text
Before:

<h1>0</h1>

After render:

<h1>1</h1>

Commit:

DOM changes from 0 → 1
```

Commit work includes things such as:

- Inserting DOM nodes
- Updating DOM properties
- Removing DOM nodes
- Attaching refs
- Running layout effects

---

## Render vs Commit — Interview Difference

| Render Phase | Commit Phase |
|---|---|
| Determines what UI should look like | Applies changes |
| Components execute | DOM is mutated |
| Reconciliation happens | DOM updates happen |
| Should be pure | Can perform DOM-related work |
| Can potentially be interrupted | Once started, the DOM mutation portion is not treated as interruptible work |
| Calculates changes | Applies changes |

##### Interview Answer

> **The render phase is where React calls components and performs reconciliation to determine what changes are needed. The commit phase is where React applies those changes to the host environment, such as the DOM.**

---

## What is Reconciliation?

Reconciliation is React's process of comparing:

```text
Previous React tree
        ↓
New React tree
        ↓
Determine differences
```

Example:

Previous:

```jsx
<div>
  <h1>Hello</h1>
  <p>Welcome</p>
</div>
```

New:

```jsx
<div>
  <h1>Hello World</h1>
  <p>Welcome</p>
</div>
```

React determines:

```text
div → same
h1 → same
h1 text → changed
p → same
```

Only the necessary change is committed.

---

## Virtual DOM

The **Virtual DOM** is a common way of describing React's in-memory representation of the UI.

Instead of directly manipulating the browser DOM every time state changes, React creates and works with JavaScript representations of the UI.

Example JSX:

```jsx
<h1>Hello</h1>
```

Conceptually produces a React element like:

```js
{
  type: "h1",
  props: {
    children: "Hello"
  }
}
```

This is **not the actual DOM node**.

It is a lightweight description of what React wants the UI to look like.

Think:

```text
React element
      ↓
"description"
      ↓
React reconciles it
      ↓
Real DOM
```

---

## Real DOM vs Virtual Representation

#### Real DOM

```js
document.querySelector("h1");
```

This gives you an actual browser DOM node.

### React Element

```jsx
<h1>Hello</h1>
```

This represents the desired UI.

Think:

```text
React element
      ↓
Desired UI description
      ↓
Reconciliation
      ↓
Real DOM
```

---

## Why Does React Use This Representation?

React wants to separate:

> **What the UI should be**

from:

> **How that UI is actually updated**

For example, React can target:

```text
Browser DOM
React Native
Other rendering environments
```

The React component describes the UI, while the renderer handles the platform-specific implementation.

---

## Is Virtual DOM Faster Than DOM?

Be careful with this interview question.

Don't say:

> "Virtual DOM is always faster than DOM."

That's incorrect.

A better answer:

> **The Virtual DOM is not inherently faster than the real DOM. React uses an in-memory representation and reconciliation to efficiently determine which host changes are necessary, avoiding unnecessary DOM operations.**

Actual performance depends on:

- Component structure
- Amount of rendering work
- Reconciliation
- DOM mutations
- Browser layout and paint
- Application architecture

---


## Fiber

**Fiber is React's internal architecture for representing and processing rendering work.**

Think of Fiber as a data structure that allows React to keep track of:

- Component instances
- Relationships between components
- Pending updates
- Work that needs to be performed
- Priority
- Reconciliation information

A Fiber node roughly represents a unit of work associated with a component or element.

---

## Why Was Fiber Introduced?

Older React rendering architecture was more synchronous.

Large rendering work could potentially block the main thread.

Imagine:

```text
User clicks button
       ↓
Huge React tree
       ↓
React performs lots of work
       ↓
Browser can't respond smoothly
```

Fiber was designed to make rendering work more flexible.

It enables React to:

- Break work into units
- Prioritize work
- Pause/restart work
- Abandon unnecessary work
- Support concurrent rendering capabilities

---

## Fiber as a Unit of Work

Think:

```text
Large rendering task
       ↓
Fiber A
Fiber B
Fiber C
Fiber D
Fiber E
```

React can process these units as part of its scheduling/reconciliation system.

Conceptually:

```text
Start work
   ↓
Fiber A
   ↓
Fiber B
   ↓
Need to prioritize something else?
   ↓
Pause/reprioritize work
   ↓
Resume appropriate work
```

This helps React remain responsive.

---

## Important: Fiber ≠ DOM

A common interview mistake:

> "Fiber is the Virtual DOM."

Not exactly.

Better:

> **Fiber is React's internal data structure and architecture used to represent units of work and manage reconciliation and scheduling. The Virtual DOM is a conceptual term for React's in-memory representation of the UI.**

They are related but not identical.

---

## Fiber Tree

React maintains an internal tree of Fiber nodes.

For:

```jsx
function App() {
  return (
    <div>
      <Header />
      <Content />
    </div>
  );
}
```

Conceptually:

```text
App Fiber
   │
   └── div Fiber
        ├── Header Fiber
        └── Content Fiber
```

Each Fiber contains information React needs to process that part of the tree.

---

## Current Tree and Work-In-Progress Tree

This is a more senior-level concept.

React can maintain:

```text
Current Fiber Tree
```

and build:

```text
Work-In-Progress Fiber Tree
```

Conceptually:

```text
Current Tree
     │
     │ update
     ↓
Work-In-Progress Tree
     │
     │ render/reconcile
     ↓
Finished work
     │
     ↓
Commit
     │
     ↓
New Current Tree
```

This allows React to prepare the next version before committing it.

---

## Render Phase + Fiber

Fiber makes the render/reconciliation process more flexible.

Conceptually:

```text
State update
     ↓
Schedule work
     ↓
Fiber reconciliation
     ↓
Render phase
     ↓
Work-in-progress tree
     ↓
Finished tree
     ↓
Commit phase
     ↓
DOM
```

The render phase can be interrupted/restarted in concurrent rendering scenarios.

---

## Concurrent Rendering

Concurrent rendering does **not** mean:

> "React uses multiple threads."

Usually, React still runs JavaScript on the browser's main thread.

Instead, it means React can manage rendering work in a way that allows higher-priority work to take precedence.

Example:

```text
Low priority:
Render huge list
       ↓
       React

High priority:
User types in input
       ↓
React prioritizes responsiveness
```

The goal is better responsiveness.

---

##  Important: Render Can Be Interrupted

With modern React's concurrent capabilities, React can:

```text
Start rendering
      ↓
Do some work
      ↓
Pause
      ↓
Handle higher-priority work
      ↓
Continue / restart / discard previous work
```

This is one reason render logic must be pure.

Imagine:

```jsx
function Component() {
  sendPayment();

  return <div />;
}
```

If React rendering were restarted, the payment could potentially be triggered multiple times.

Therefore:

> **Side effects should not happen during render.**

Use effects when the side effect truly belongs there:

```jsx
useEffect(() => {
  sendPayment();
}, []);
```

---

##  Full React Rendering Flow

Suppose:

```jsx
function Counter() {
  const [count, setCount] = useState(0);

  return <h1>{count}</h1>;
}
```

User clicks:

```text
setCount(1)
       ↓
React schedules update
       ↓
      Batching
       ↓
Fiber identifies work
       ↓
Render Phase
       ↓
Counter executes
       ↓
New React element
<h1>1</h1>
       ↓
Reconciliation
       ↓
React determines:
"text changed from 0 → 1"
       ↓
Commit Phase
       ↓
DOM updated
       ↓
Browser paints
```

That's the core React mental model.

---

## Complete Mental Model

```text
                 STATE / PROPS CHANGE
                         │
                         ↓
                  Update scheduled
                         │
                         ↓
                     Batching
                         │
                         ↓
                 React Render Phase
                         │
                 ┌───────┴────────┐
                 ↓                ↓
             Components       Reconciliation
                 │                │
                 └───────┬────────┘
                         ↓
                   Fiber Work
                         │
                         ↓
                 New UI calculated
                         │
                         ↓
                  Commit Phase
                         │
                         ↓
                  DOM mutations
                         │
                         ↓
                  Browser Paint
                         │
                         ↓
                Passive Effects
                   useEffect
```

---

## Final Revision Sheet

| Concept | Remember This |
|---|---|
| Rendering | React calculates what UI should look like |
| Re-render | Component function executes again |
| DOM update | Happens during commit when necessary |
| Render phase | Calculate/reconcile |
| Commit phase | Apply changes |
| Reconciliation | Compare previous and next UI |
| Virtual DOM | In-memory UI representation/concept |
| Fiber | Internal architecture/data structure |
| Fiber node | Unit of React work |
| Batching | Group multiple updates |
| Functional update | `setState(prev => ...)` |
| Concurrent rendering | React can manage rendering work more flexibly |
| Render purity | No side effects during render |
| `useEffect` | Passive effects after commit/typically after paint |
| `useLayoutEffect` | Runs during commit before browser paint |

---

## What to Memorize for Interviews

The core flow:

```text
1. State / Props change
        ↓
2. React schedules work
        ↓
3. Batching may group updates
        ↓
4. Render phase
        ↓
5. Reconciliation using Fiber
        ↓
6. Commit phase
        ↓
7. DOM mutations
        ↓
8. Browser paint
        ↓
9. Passive effects
```

