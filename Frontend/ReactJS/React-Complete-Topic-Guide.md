# React Interview — Complete Topic Guide

> Every major React interview topic and subtopic with an interview-friendly explanation, code, and a scenario-based example.
>

**How each subtopic is laid out**
- **Explanation**: what to say in the interview
- **Code**: the smallest example that proves you understand it
- **Scenario**: a "what would you do if…" question and the answer
- **Tip / Follow-up**: what interviewers dig into next

---

## Table of Contents

1. [React Fundamentals](#1-react-fundamentals)
2. [Rendering & Reconciliation](#2-rendering--reconciliation)
3. [Hooks](#3-hooks)
4. [Class Components (Legacy)](#4-class-components-legacy)
5. [Component Patterns](#5-component-patterns)
6. [State Management](#6-state-management)
7. [Performance Optimization](#7-performance-optimization)
8. [Concurrent React (React 18+)](#8-concurrent-react-react-18)
9. [React 19](#9-react-19)
10. [Rendering Strategies & SSR](#10-rendering-strategies--ssr)
11. [Routing](#11-routing)
12. [Forms](#12-forms)
13. [Error Handling & Resilience](#13-error-handling--resilience)
14. [Testing](#14-testing)
15. [Styling](#15-styling)
16. [TypeScript with React](#16-typescript-with-react)
17. [Security](#17-security)
18. [Accessibility (a11y)](#18-accessibility-a11y)
19. [Architecture (Senior / Staff)](#19-architecture-senior--staff)
20. [JavaScript Fundamentals](#20-javascript-fundamentals)
21. [Machine Coding / Live Coding](#21-machine-coding--live-coding)
22. [Behavioural & Experience](#22-behavioural--experience)

---

## 1. React Fundamentals

### 1.1 What is React? Declarative vs Imperative, Library vs Framework

**Explanation**
React is a **JavaScript library for building user interfaces** out of components. Its core idea: **UI = f(state)**. You describe *what* the UI should look like for a given state, and React works out *how* to update the DOM.

- **Imperative** (vanilla JS / jQuery): you write the steps yourself: find the element, change its text, add a class.
- **Declarative** (React): you describe the result; React handles the DOM operations.

React is called a **library**, not a framework, because it only handles the view layer. Routing, data fetching, forms and build tooling are your choice (React Router, React Query, Vite…). **Next.js / Remix** are frameworks built *on* React.

```js
// Imperative
const btn = document.querySelector('#like');
btn.addEventListener('click', () => {
  liked = !liked;
  btn.textContent = liked ? 'Unlike' : 'Like';
  btn.classList.toggle('active', liked);
});
```
```jsx
// Declarative
function LikeButton() {
  const [liked, setLiked] = useState(false);
  return (
    <button className={liked ? 'active' : ''} onClick={() => setLiked(l => !l)}>
      {liked ? 'Unlike' : 'Like'}
    </button>
  );
}
```

**Scenario:** *"Why would a team move from jQuery to React?"*
In jQuery, as the UI grows, many places change the DOM directly and the state is spread across DOM nodes, so it gets out of sync. React keeps state in one place and re-derives the UI from it, so the UI is predictable, components are reusable, and the code is easier to test.

**Tip:** Mention the trade-off: React gives flexibility but needs ecosystem decisions. Frameworks like Next.js make those decisions for you.

---

### 1.2 JSX and What It Compiles To

**Explanation**
JSX is **syntax sugar** for function calls that create plain JS objects called **React elements**. Browsers don't understand JSX; Babel/SWC/esbuild compile it.

- The old transform: `React.createElement(type, props, ...children)`, which needed `import React` in every file.
- The **new JSX transform** (React 17+): `jsx()` from `react/jsx-runtime`, auto-imported.

```jsx
const el = <h1 className="title">Hello {name}</h1>;

// compiles to (new transform)
import { jsx as _jsx } from 'react/jsx-runtime';
const el = _jsx('h1', { className: 'title', children: ['Hello ', name] });

// which produces a plain object (React element)
// { type: 'h1', props: { className: 'title', children: [...] }, key: null, ... }
```

JSX rules:
- Return **one root** (or a Fragment).
- `className` not `class`, `htmlFor` not `for`, camelCase events (`onClick`).
- `{}` embeds any **expression** (not statements: no `if`/`for` inside; use ternary/`&&`/`map`).
- Lowercase tag = DOM element (`'div'`); Capitalized = component (`MyComp`).
- JSX **escapes values** by default → protects against XSS.

**Scenario:** *"Why must component names start with a capital letter?"*
Because the compiler uses case to decide: `<button>` becomes the string `'button'`, while `<Button>` becomes a reference to the `Button` variable. A lowercase component would be treated as an unknown HTML tag.

**Follow-up:** *"Is JSX required?"* No, you can call `createElement` directly. JSX just makes it readable.

---

### 1.3 Components: Function vs Class

**Explanation**
A component is a function (or class) that takes **props** and returns **React elements**.

| | Function component | Class component |
|---|---|---|
| State | `useState`, `useReducer` | `this.state`, `this.setState` |
| Side effects | `useEffect` | lifecycle methods |
| `this` binding | none | needed (`bind` or arrow methods) |
| Logic reuse | custom hooks | HOCs / render props |
| Status | **recommended** | legacy (still needed for Error Boundaries) |

```jsx
// Function
function Greeting({ name }) {
  return <p>Hello, {name}</p>;
}

// Class
class Greeting extends React.Component {
  render() {
    return <p>Hello, {this.props.name}</p>;
  }
}
```

**Scenario:** *"Your codebase has 200 class components. Do you rewrite them all to hooks?"*
No. Class components still work and get support. Migrate step by step: write new code with hooks, and convert a class only when you're already changing it for a feature or bug. Add tests before converting so the behaviour stays the same.

---

### 1.4 Props vs State & One-Way Data Flow

**Explanation**
- **Props**: inputs passed from parent → child. **Read-only** in the child.
- **State**: data owned by the component that changes over time; updating it triggers a re-render.
- **One-way data flow**: data goes **down** via props; changes go **up** via callbacks. This makes it easy to trace where a value comes from.

```jsx
function Parent() {
  const [count, setCount] = useState(0);            // state lives here
  return <Child count={count} onIncrement={() => setCount(c => c + 1)} />;
}

function Child({ count, onIncrement }) {           // props are read-only
  return <button onClick={onIncrement}>Clicked {count} times</button>;
}
```

**Scenario:** *"A child needs to change data shown in its sibling. How?"*
Lift the state up to the closest common parent, pass the value down to the sibling, and pass a setter callback down to the child. If they're far apart in the tree, consider Context or a store.

**Tip:** Never mutate props (`props.user.name = 'x'`). This breaks the parent's state and memoisation.

**Follow-up:** *"Should I copy props into state?"* Usually no. If you write `useState(props.value)`, the state is set **only once** and won't update when the prop changes. Derive the value during render instead, or use a `key` to reset the component.

---

### 1.5 Rendering Lists & Keys

**Explanation**
`key` tells React **which item is which** between renders, so it can reuse, reorder or remove the right DOM nodes and component state.

- Keys must be **stable, unique among siblings, and predictable**: use database IDs.
- **Index as key** is fine only if the list never reorders, filters or inserts. Otherwise, state (input values, checkbox, focus) sticks to the *position*, not the *item*.
- `Math.random()` as key is the worst: every render remounts every item.

```jsx
// ❌ index key: type in row 1, then delete row 0 → your text moves to the wrong item
{todos.map((t, i) => <TodoRow key={i} todo={t} />)}

// ✅ stable id
{todos.map(t => <TodoRow key={t.id} todo={t} />)}
```

**Scenario:** *"Users report that after deleting an item, the input text appears in the wrong row."*
That's the classic index-as-key bug. `TodoRow` has local state (or an uncontrolled input). After deleting row 0, the old row 1 is now at index 0, so React reuses the index-0 component and its state. Fix: use `key={todo.id}`.

**Bonus trick:** Changing the key **remounts** the component and resets its state:
```jsx
<ProfileForm key={userId} userId={userId} />   // switching user resets the form
```

---

### 1.6 Conditional Rendering

**Explanation**
Use normal JS expressions: ternary, `&&`, early return, or an object map.

```jsx
function Status({ loading, error, items }) {
  if (loading) return <Spinner />;                       // early return
  if (error) return <ErrorMsg error={error} />;

  return (
    <>
      {items.length > 0 ? <List items={items} /> : <Empty />}   {/* ternary */}
      {items.length > 100 && <p>Showing first 100</p>}          {/* && */}
    </>
  );
}

// Object map for many variants
const icons = { success: <Check />, error: <Cross />, warning: <Alert /> };
return icons[type] ?? null;
```

**Scenario (gotcha):** *"Why does my UI show a `0`?"*
```jsx
{items.length && <List />}   // when length is 0 → renders "0"
```
`&&` returns the left side when it's falsy, and React renders the number `0` (but not `false`/`null`/`undefined`). Fix: `{items.length > 0 && <List />}` or `!!items.length`.

**Tip:** Returning `null` renders nothing, but the component still runs its hooks.

---

### 1.7 Fragments

**Explanation**
Fragments let you return several elements without adding an extra DOM node. That matters for CSS layouts (flex/grid) and valid HTML (`<tr>` inside `<tbody>`).

```jsx
function Columns() {
  return (
    <>
      <td>Name</td>
      <td>Age</td>
    </>
  );
}

// Need a key? Use the long form
{items.map(item => (
  <React.Fragment key={item.id}>
    <dt>{item.term}</dt>
    <dd>{item.desc}</dd>
  </React.Fragment>
))}
```

**Scenario:** *"A `<div>` wrapper is breaking a table layout."* Return a Fragment, since `<div>` isn't allowed inside `<tr>`.

---

### 1.8 Controlled vs Uncontrolled Components

**Explanation**
- **Controlled**: React state is the single source of truth. `value` + `onChange`.
- **Uncontrolled**: the DOM keeps the value; you read it via `ref` or `FormData` when needed. `defaultValue` sets the initial value.

```jsx
// Controlled
function Controlled() {
  const [email, setEmail] = useState('');
  const invalid = email && !email.includes('@');
  return (
    <>
      <input value={email} onChange={e => setEmail(e.target.value)} />
      {invalid && <span>Invalid email</span>}
    </>
  );
}

// Uncontrolled
function Uncontrolled() {
  const ref = useRef(null);
  const submit = e => { e.preventDefault(); console.log(ref.current.value); };
  return (
    <form onSubmit={submit}>
      <input defaultValue="" ref={ref} />
      <button>Save</button>
    </form>
  );
}
```

| Use controlled when | Use uncontrolled when |
|---|---|
| Validation as you type, input masking | Simple forms, read once on submit |
| Value depends on other state | Performance-heavy large forms (React Hook Form) |
| Disable submit based on input | File inputs (`<input type="file">` is always uncontrolled) |

**Scenario:** *"Warning: A component is changing an uncontrolled input to be controlled."*
The `value` started as `undefined` and later became a string. Fix: always set an initial value, e.g. `useState('')`, or `value={name ?? ''}`.

---

### 1.9 Lifting State Up, Prop Drilling & Composition

**Explanation**
- **Lifting state up**: move state to the closest common parent of the components that need it.
- **Prop drilling**: passing props through components that don't use them, only to reach a deep child. It's annoying and makes components tightly coupled.
- Fixes, in order of preference: **composition (`children`)** → **Context** → **state library**.

```jsx
// ❌ Drilling: Layout and Sidebar don't need `user`
<Layout user={user}>  →  <Sidebar user={user}>  →  <Avatar user={user} />

// ✅ Composition: build the deep element where the data is
function App() {
  const user = useUser();
  return (
    <Layout sidebar={<Sidebar avatar={<Avatar user={user} />} />}>
      <Main />
    </Layout>
  );
}
function Layout({ sidebar, children }) {
  return <div className="grid">{sidebar}{children}</div>;
}
```

**Scenario:** *"You're passing `theme` through 6 levels."*
First ask whether the in-between components could take `children` instead. If many unrelated deep components need it (theme, locale, auth), that's a good fit for Context.

---

### 1.10 Synthetic Events & Event Handling

**Explanation**
React wraps native events in a **SyntheticEvent**, which works the same in every browser (`e.preventDefault()`, `e.stopPropagation()`, `e.target`).

- **Event delegation**: React attaches one listener per event type at the **root container** (since React 17; before that, on `document`).
- Event pooling was **removed in React 17**, so you can read `e` asynchronously now.
- Pass a **function**, don't call it: `onClick={handle}` not `onClick={handle()}`.
- `e.nativeEvent` gives you the browser event.

```jsx
function Item({ id, onDelete }) {
  const handleClick = (e) => {
    e.stopPropagation();          // don't trigger the parent row's onClick
    onDelete(id);
  };
  return (
    <li onClick={() => console.log('row clicked')}>
      Item {id}
      <button onClick={handleClick}>Delete</button>
    </li>
  );
}
```

**Scenario:** *"Clicking Delete also opens the row's detail page."* The event bubbles up to the row. Call `e.stopPropagation()` in the button handler.

**Follow-up:** *"Why was the change from document to root in React 17 important?"* It lets several React versions run on the same page (micro-frontends, gradual upgrades) without their event systems interfering with each other.

---

### 1.11 Pure Components & Keeping Render Pure

**Explanation**
A render must be a **pure function** of props + state + context:
- Same input → same JSX output.
- **No side effects during render**: no API calls, no mutating outside variables, no `setState` on other components, no DOM changes.

React may call render **more than once**: StrictMode double-renders in dev, and concurrent rendering can throw a render away and start again. Side effects in render would then run several times.

```jsx
// ❌ Impure: mutates external variable
let count = 0;
function Bad() { count++; return <p>{count}</p>; }

// ❌ Impure: mutates props
function Bad2({ items }) { items.sort(); return <List items={items} />; }

// ✅ Pure: copy before sorting
function Good({ items }) {
  const sorted = [...items].sort((a, b) => a.name.localeCompare(b.name));
  return <List items={sorted} />;
}
```

**Scenario:** *"An analytics event fires twice in development."* It was called during render, or in an effect without proper handling. StrictMode exposes this. Move it to an event handler (for user actions) or an effect (for "screen viewed"), and make it idempotent (safe to run twice).

---

## 2. Rendering & Reconciliation

### 2.1 Virtual DOM & the Diffing Algorithm

**Explanation**
The Virtual DOM is a lightweight **JS object tree** describing the UI. On every update:
1. React builds a new element tree.
2. It **diffs** the new tree against the previous one (**reconciliation**).
3. It applies only the minimal changes to the real DOM.

A general tree diff costs O(n³). React gets **O(n)** by using two heuristics:
1. **Different element type → throw away the whole subtree** and rebuild it (`<div>` → `<span>`, or `ComponentA` → `ComponentB`).
2. **Keys** identify children across renders within a list.

```jsx
// Type changes → <Counter> is UNMOUNTED and its state LOST
{isAdmin ? <div><Counter /></div> : <section><Counter /></section>}

// Same type at same position → state PRESERVED, only props update
{isAdmin ? <Counter color="red" /> : <Counter color="blue" />}
```

**Scenario:** *"Toggling a layout wrapper resets a form's state."* The wrapper's element type changed, so React remounted the whole subtree. Keep the same wrapper type and change only its props or className.

**Tip:** "Virtual DOM is fast" is a myth. The real benefit is that you get **declarative code with good-enough performance**, without hand-writing DOM updates. Svelte and Solid skip the VDOM altogether.

---

### 2.2 Reconciliation Heuristics: Position, Type, Key

**Explanation**
React identifies a component by **its type + its position in the tree (+ key)**. State is tied to that position, not to the component's variable name.

```jsx
// Same position, same type → state is SHARED across the toggle
function Scoreboard() {
  const [isPlayerA, setIsPlayerA] = useState(true);
  return (
    <>
      {isPlayerA ? <Counter person="A" /> : <Counter person="B" />}
      <button onClick={() => setIsPlayerA(p => !p)}>Switch</button>
    </>
  );
}

// Fix 1: different keys → separate state
{isPlayerA ? <Counter key="A" person="A" /> : <Counter key="B" person="B" />}

// Fix 2: different positions
{isPlayerA && <Counter person="A" />}
{!isPlayerA && <Counter person="B" />}
```

**Scenario:** *"Never define components inside other components. Why?"*
```jsx
function Parent() {
  function Child() { /* ... */ }   // new function identity every render
  return <Child />;
}
```
Every render creates a **new component type**, so React sees a different type and remounts `Child`, losing its state and focus each time. Define components at module level.

---

### 2.3 What Triggers a Re-render

**Explanation**
A component re-renders when:
1. **Its state changes**: `setState` with a value that's different by `Object.is`.
2. **Its parent re-renders**: by default *all* children re-render, whether or not their props changed.
3. **A context it consumes changes**.
4. (A subscribed external store changes, via `useSyncExternalStore`.)

**Props changing doesn't trigger a re-render on its own.** Props only change because the parent re-rendered.

```jsx
function Parent() {
  const [n, setN] = useState(0);
  return (
    <>
      <button onClick={() => setN(n + 1)}>{n}</button>
      <ExpensiveChild />   {/* re-renders on every click even with no props! */}
    </>
  );
}
```

Ways to avoid it:
```jsx
// 1. Move state down: isolate the changing part
function Counter() { const [n, setN] = useState(0); return <button onClick={() => setN(n + 1)}>{n}</button>; }
function Parent() { return <><Counter /><ExpensiveChild /></>; }

// 2. Pass as children: the element is created by the grandparent, so it's referentially stable
function ScrollTracker({ children }) {
  const [y, setY] = useState(0);
  // ...updates y on scroll
  return <div>{y}{children}</div>;  // children won't re-render when y changes
}

// 3. React.memo
const ExpensiveChild = React.memo(function ExpensiveChild() { /* ... */ });
```

**Scenario:** *"Setting state to the same value: does it re-render?"*
If `Object.is(old, new)` is true, React **bails out** (skips the render). It may still render that component once more before bailing out, but it won't re-render the children or run effects. Note: `setUser({...user})` creates a **new object**, so it always re-renders.

---

### 2.4 Render Phase vs Commit Phase

**Explanation**
| Render phase | Commit phase |
|---|---|
| React calls your components, builds the new tree, diffs it | React applies the DOM changes |
| **Pure**; no side effects | Side effects allowed |
| Can be **paused, aborted, restarted** (concurrent) | **Synchronous**; can't be interrupted |
| — | Order: DOM mutations → refs attached → `useLayoutEffect` → browser paints → `useEffect` |

```jsx
function Demo() {
  console.log('render');                         // render phase
  useLayoutEffect(() => console.log('layout'));  // commit, before paint
  useEffect(() => console.log('effect'));        // after paint
  return <div ref={() => console.log('ref')} />;
}
// Logs: render → ref → layout → effect
```

**Scenario:** *"Why can't I fetch data in the component body?"* The body runs during the render phase. It may run several times, and calling setState there causes infinite loops. Fetch in an effect, a data library, or a Server Component.

---

### 2.5 React Fiber

**Explanation**
Fiber (React 16) is the **rewrite of the reconciler**. Each component instance is a **fiber node**: a JS object with `type`, `props`, `stateNode`, `child`/`sibling`/`return` pointers, `memoizedState` (the hooks list), `lanes` (priority) and effect flags.

Why it matters:
- The old "stack reconciler" walked the tree recursively and **couldn't stop** until it finished, so big updates blocked the main thread and dropped frames.
- Fiber turns the tree walk into a **loop over linked nodes**, so React can **pause, prioritise, resume or abandon** work.
- **Double buffering**: there's a `current` tree (on screen) and a `workInProgress` tree (being built). On commit, they swap.
- **Lanes** give priorities: user input (sync) > transitions > idle work.

```
App (fiber)
 └─child→ Header ─sibling→ Main ─sibling→ Footer
                            └─child→ List
          (each node has .return → parent)
```

**Scenario:** *"Typing in a search box lags while a 5,000-item list re-renders."*
Fiber makes **concurrent features** possible. Wrap the list update in `startTransition` (or use `useDeferredValue`). React then treats typing as urgent and interrupts the list render whenever a new keystroke comes in.

**Tip:** You don't use Fiber directly. Explain it as **the thing that makes interruptible rendering, Suspense and transitions possible**.

---

### 2.6 Batching (Automatic Batching in React 18)

**Explanation**
**Batching** means grouping several state updates into **one re-render**.

- **React 17**: batched only inside React event handlers. Updates in `setTimeout`, promises or native listeners each caused their own render.
- **React 18 (`createRoot`)**: **automatic batching everywhere**.
- Opt out with `flushSync` (rarely needed, e.g. to measure the DOM right after an update).

```jsx
function handleClick() {
  setCount(c => c + 1);
  setFlag(f => !f);
  // React 17 & 18: ONE render
}

fetch('/api').then(() => {
  setCount(c => c + 1);
  setFlag(f => !f);
  // React 17: TWO renders · React 18: ONE render
});

import { flushSync } from 'react-dom';
flushSync(() => setItems(next));    // DOM updated synchronously here
listRef.current.lastChild.scrollIntoView();
```

**Scenario:** *"After `setCount(count + 1)`, `console.log(count)` shows the old value."*
State updates are **queued**, not applied straight away. `count` is a constant in the current render's closure. The new value shows up on the next render. If you need the next value now, compute it into a variable first: `const next = count + 1; setCount(next); use(next);`.

---

### 2.7 StrictMode

**Explanation**
`<React.StrictMode>` is a **development-only** tool that surfaces bugs early:
- **Renders components twice** → catches impure renders.
- **Runs effects mount → unmount → mount** → catches missing cleanups.
- Warns about deprecated APIs (string refs, `findDOMNode`, legacy lifecycles).
- It has **no effect in production**.

```jsx
// ❌ Without cleanup: StrictMode reveals two subscriptions in dev
useEffect(() => {
  socket.on('message', handler);
}, []);

// ✅
useEffect(() => {
  socket.on('message', handler);
  return () => socket.off('message', handler);
}, []);
```

**Scenario:** *"My API is called twice on page load, so I removed StrictMode."*
That's the wrong fix. The double call only happens in dev and shows that your effect doesn't handle being re-run. Use an `AbortController` (or an `ignore` flag) in the cleanup, or better, use a data-fetching library that de-duplicates requests. Production calls it once.

---

## 3. Hooks

### 3.1 `useState`

**Explanation**
`const [state, setState] = useState(initial)` adds local state. Key points:
- **Functional updates** `setX(prev => ...)` when the next value depends on the previous one. They're safe with batching and stale closures.
- **Lazy initialisation** `useState(() => expensive())`: the function runs only on the first render.
- **Immutability**: always create new objects/arrays; React compares with `Object.is`.
- Each render has its own **snapshot** of state.

```jsx
// Functional updates
setCount(count + 1); setCount(count + 1);   // +1 (both use the same snapshot)
setCount(c => c + 1); setCount(c => c + 1); // +2

// Lazy init: reads localStorage only once
const [todos, setTodos] = useState(() => JSON.parse(localStorage.getItem('todos')) ?? []);

// ❌ useState(JSON.parse(localStorage.getItem('todos')))  → parses on EVERY render

// Immutable updates
setUser(u => ({ ...u, address: { ...u.address, city: 'Pune' } }));
setTodos(t => t.map(x => x.id === id ? { ...x, done: !x.done } : x));
setTodos(t => t.filter(x => x.id !== id));
setTodos(t => [...t, newTodo]);
```

**Scenario:** *"I push to an array in state but the UI doesn't update."*
```jsx
todos.push(item); setTodos(todos);   // same reference → React bails out
```
React sees the same array reference and skips the render. Fix: `setTodos([...todos, item])`.

**Follow-up:** *"Why is the state update async?"* It isn't really "async" like a promise. Updates are **queued and batched**, then applied on the next render. This avoids extra renders and keeps the UI consistent within one render.

---

### 3.2 `useEffect`

**Explanation**
`useEffect` **syncs your component with an external system** (network, subscriptions, timers, DOM APIs, analytics) **after** React paints.

| Dependency array | Runs |
|---|---|
| none | after every render |
| `[]` | after the first render (mount) |
| `[a, b]` | after mount and whenever `a` or `b` changes (`Object.is`) |

**Cleanup** (the function you return) runs **before the effect runs again** and **on unmount**.

```jsx
function ChatRoom({ roomId }) {
  useEffect(() => {
    const conn = createConnection(roomId);
    conn.connect();
    return () => conn.disconnect();   // runs before reconnecting to the new room
  }, [roomId]);
}
```

**Race conditions in fetching** (asked very often):
```jsx
useEffect(() => {
  let ignore = false;
  fetch(`/api/users/${id}`)
    .then(r => r.json())
    .then(data => { if (!ignore) setUser(data); });
  return () => { ignore = true; };     // a stale response is ignored
}, [id]);

// or abort the request entirely
useEffect(() => {
  const ctrl = new AbortController();
  fetch(`/api/users/${id}`, { signal: ctrl.signal }).then(r => r.json()).then(setUser)
    .catch(e => { if (e.name !== 'AbortError') setError(e); });
  return () => ctrl.abort();
}, [id]);
```

**Scenario:** *"User clicks profile 1 then quickly profile 2, but sees profile 1's data."*
Request 1 came back after request 2 and overwrote it. Fix with the `ignore` flag or `AbortController` above. In production, use React Query / SWR, which handle this for you.

**Common mistakes**
```jsx
// ❌ Infinite loop: object dep is new every render
useEffect(() => { fetchData(options); }, [options]);   // options = { page } created in render
// ✅ depend on primitives
useEffect(() => { fetchData({ page }); }, [page]);

// ❌ async effect function (returns a promise, not a cleanup)
useEffect(async () => { ... }, []);
// ✅
useEffect(() => { (async () => { ... })(); }, []);
```

---

### 3.3 "You Might Not Need an Effect"

**Explanation**
Effects are for syncing with **external** systems. Many effects in real code shouldn't be effects at all:

```jsx
// ❌ Derived state via effect: extra render, can go out of sync
const [fullName, setFullName] = useState('');
useEffect(() => setFullName(`${first} ${last}`), [first, last]);
// ✅ compute during render
const fullName = `${first} ${last}`;

// ❌ Expensive derived data in effect
// ✅ useMemo
const visible = useMemo(() => todos.filter(t => matches(t, filter)), [todos, filter]);

// ❌ Event logic in effect
useEffect(() => { if (submitted) post('/api', form); }, [submitted]);
// ✅ do it in the handler
function handleSubmit() { post('/api', form); }

// ❌ Resetting state when a prop changes
useEffect(() => { setComment(''); }, [userId]);
// ✅ key resets the component
<Profile key={userId} userId={userId} />
```

**Scenario:** *"Reviewing a PR with 6 `useEffect`s that set state from other state."*
Point out that each one adds an extra render and can get out of sync. Replace them with values derived during render, `useMemo`, logic in event handlers, or a `key` reset.

---

### 3.4 `useLayoutEffect` vs `useEffect`

**Explanation**
- `useEffect`: runs **after paint**, without blocking it. Use it for most side effects.
- `useLayoutEffect`: runs **synchronously after DOM mutation, before paint**. Use it when you must **measure the layout and change it** before the user sees anything (to avoid flicker).
- Overusing `useLayoutEffect` blocks painting and hurts performance. It also does nothing on the server (React warns during SSR).

```jsx
function Tooltip({ targetRect, children }) {
  const ref = useRef(null);
  const [top, setTop] = useState(0);

  useLayoutEffect(() => {
    const { height } = ref.current.getBoundingClientRect();
    // place above target if there's room, otherwise below; no visible jump
    setTop(targetRect.top - height > 0 ? targetRect.top - height : targetRect.bottom);
  }, [targetRect]);

  return <div ref={ref} style={{ position: 'absolute', top }}>{children}</div>;
}
```

**Scenario:** *"A tooltip flashes at the wrong position before jumping to the right one."* The measurement happens in `useEffect`, after paint. Moving it to `useLayoutEffect` fixes the flicker.

**Also:** `useInsertionEffect` runs before layout effects and exists for CSS-in-JS libraries to inject styles. You'll almost never use it yourself.

---

### 3.5 `useRef`

**Explanation**
`useRef(initial)` returns a **mutable object `{ current }`** that persists for the component's lifetime. **Changing `.current` does not re-render.**

Uses:
1. **DOM access**: focus, scroll, measure, media playback.
2. **Values that survive renders without being shown**: timer IDs, previous values, "is mounted", the latest callback.

```jsx
// DOM access
function SearchBox() {
  const inputRef = useRef(null);
  useEffect(() => { inputRef.current.focus(); }, []);
  return <input ref={inputRef} />;
}

// Timer ID
function Stopwatch() {
  const [ms, setMs] = useState(0);
  const timer = useRef(null);
  const start = () => { timer.current ??= setInterval(() => setMs(m => m + 10), 10); };
  const stop = () => { clearInterval(timer.current); timer.current = null; };
  useEffect(() => stop, []);                  // clear on unmount
  return <><p>{ms}</p><button onClick={start}>Start</button><button onClick={stop}>Stop</button></>;
}

// Previous value
function usePrevious(value) {
  const ref = useRef();
  useEffect(() => { ref.current = value; }, [value]);
  return ref.current;           // returns the value from the previous render
}
```

**ref vs state**
| | `useState` | `useRef` |
|---|---|---|
| Triggers re-render | Yes | No |
| Used in JSX output | Yes | No (don't read `.current` during render) |
| Updated | async (next render) | immediately |

**Scenario:** *"Store a timer ID in a normal `let` variable?"* No. It's reset on every render. Use `useRef`.

---

### 3.6 `useReducer`

**Explanation**
`const [state, dispatch] = useReducer(reducer, initialState)`. It puts state transitions in one **pure function**.

Prefer it over `useState` when:
- State has several related fields that change together (`status`, `data`, `error`).
- The next state depends on the previous one in complex ways.
- You want transition logic that's easy to test and can be pulled out of the component.
- `dispatch` is **stable**, so it's safe to pass down without `useCallback`.

```jsx
const initial = { status: 'idle', data: null, error: null };

function reducer(state, action) {
  switch (action.type) {
    case 'FETCH_START':   return { ...state, status: 'loading', error: null };
    case 'FETCH_SUCCESS': return { status: 'success', data: action.payload, error: null };
    case 'FETCH_ERROR':   return { ...state, status: 'error', error: action.error };
    default: throw new Error(`Unknown action ${action.type}`);
  }
}

function Users() {
  const [state, dispatch] = useReducer(reducer, initial);
  useEffect(() => {
    dispatch({ type: 'FETCH_START' });
    getUsers()
      .then(d => dispatch({ type: 'FETCH_SUCCESS', payload: d }))
      .catch(e => dispatch({ type: 'FETCH_ERROR', error: e }));
  }, []);
  // ...
}
```

**Scenario:** *"There's a bug where `loading` is true and `error` is set at the same time."*
Separate booleans (`isLoading`, `isError`) allow combinations that should be impossible. Model it as one `status` field (a state machine) with a reducer, so those combinations can't happen.

---

### 3.7 `useContext`

**Explanation**
Context passes data deep down the tree **without prop drilling**. Good for app-wide values that change rarely: theme, locale, auth user, feature flags.

```jsx
const ThemeContext = createContext('light');

function App() {
  const [theme, setTheme] = useState('light');
  const value = useMemo(() => ({ theme, setTheme }), [theme]);   // stable value
  return (
    <ThemeContext.Provider value={value}>   {/* React 19: <ThemeContext value={value}> */}
      <Page />
    </ThemeContext.Provider>
  );
}

function ThemeToggle() {
  const { theme, setTheme } = useContext(ThemeContext);
  return <button onClick={() => setTheme(t => (t === 'light' ? 'dark' : 'light'))}>{theme}</button>;
}

// Good practice: custom hook with a guard
function useTheme() {
  const ctx = useContext(ThemeContext);
  if (!ctx) throw new Error('useTheme must be used inside ThemeProvider');
  return ctx;
}
```

**Performance pitfall:** **every consumer re-renders** when the Provider's `value` changes (by reference). `React.memo` doesn't stop it.

**Scenario:** *"The whole app re-renders on every keystroke in a form whose state lives in Context."*
Fixes:
1. **Split contexts** by how often they change (e.g. `UserContext` vs `FormContext`).
2. **Separate state and dispatch** into two contexts. Components that only dispatch won't re-render.
3. **Memoise the `value`** object.
4. Use a **store with selectors** (Zustand, Redux) for state that changes often.

---

### 3.8 `useMemo` & `useCallback`

**Explanation**
- `useMemo(() => compute(a, b), [a, b])` caches a **computed value**.
- `useCallback(fn, [deps])` caches a **function reference**. It's the same as `useMemo(() => fn, deps)`.

They help in only **two cases**:
1. **Expensive calculations** (filtering/sorting thousands of items).
2. **Referential stability**: the value is passed to a `React.memo` child, or used as an effect/hook dependency.

```jsx
function ProductList({ products, query, onSelect }) {
  // 1. expensive computation
  const filtered = useMemo(
    () => products.filter(p => p.name.toLowerCase().includes(query.toLowerCase())),
    [products, query]
  );

  // 2. stable callback for a memoised child
  const handleSelect = useCallback(id => onSelect(id), [onSelect]);

  return filtered.map(p => <ProductRow key={p.id} product={p} onSelect={handleSelect} />);
}

const ProductRow = React.memo(function ProductRow({ product, onSelect }) {
  return <li onClick={() => onSelect(product.id)}>{product.name}</li>;
});
```

**Scenario:** *"A teammate wrapped every function in `useCallback`. Good?"*
No. If the child isn't memoised, a stable function reference doesn't help, and you still pay for the hook and its dependency comparison. Memoisation has a cost (memory, comparisons, harder-to-read code). **Measure with the Profiler first.** The **React Compiler** (React 19 era) now memoises automatically, which mostly removes the need to do this by hand.

**Follow-up:** *"Does useMemo guarantee caching?"* No. React may throw the cache away. Treat it as a performance hint, never as something your logic depends on.

---

### 3.9 Other Built-in Hooks

**`useId`**: generates unique IDs that match between server and client (for accessibility attributes). Don't use it for list keys.
```jsx
function Field({ label }) {
  const id = useId();
  return <><label htmlFor={id}>{label}</label><input id={id} /></>;
}
```

**`useImperativeHandle`**: chooses what a parent can do through a ref, instead of exposing the whole DOM node.
```jsx
function VideoPlayer({ src, ref }) {           // React 19: ref as a prop
  const video = useRef(null);
  useImperativeHandle(ref, () => ({
    play: () => video.current.play(),
    pause: () => video.current.pause(),
  }), []);
  return <video ref={video} src={src} />;
}
// parent: playerRef.current.play()
```

**`useSyncExternalStore`**: safely subscribes to a store **outside React** (browser APIs, Redux, Zustand) without "tearing" (different components showing different values) during concurrent rendering.
```jsx
function useOnlineStatus() {
  return useSyncExternalStore(
    cb => { addEventListener('online', cb); addEventListener('offline', cb);
            return () => { removeEventListener('online', cb); removeEventListener('offline', cb); }; },
    () => navigator.onLine,     // client snapshot
    () => true                  // server snapshot
  );
}
```

**`useTransition` / `useDeferredValue`**: see [Section 8](#8-concurrent-react-react-18).

**`useDebugValue`**: shows a label for a custom hook in React DevTools.

---

### 3.10 Rules of Hooks (and Why)

**Explanation**
1. Call hooks **only at the top level**: never inside conditions, loops, nested functions or after an early return.
2. Call hooks **only from function components or custom hooks**.

**Why:** React doesn't know hooks by name. It stores them in a **linked list on the fiber, in the order they're called**. On every render it walks that list by position. If a hook is skipped conditionally, every hook after it reads the wrong slot.

```jsx
// ❌ Breaks order
function Profile({ userId }) {
  if (!userId) return null;          // early return BEFORE hooks
  const [user, setUser] = useState(null);
}

// ❌ conditional hook
if (isEdit) { useEffect(() => {...}); }

// ✅ condition INSIDE the hook
function Profile({ userId }) {
  const [user, setUser] = useState(null);
  useEffect(() => {
    if (!userId) return;
    fetchUser(userId).then(setUser);
  }, [userId]);
  if (!userId) return null;
  // ...
}
```

**Scenario:** *"Error: Rendered more hooks than during the previous render."* A hook is called conditionally, or after an early return. Move every hook above any `return`.

**Tool:** `eslint-plugin-react-hooks` (`rules-of-hooks` + `exhaustive-deps`) catches this. Always keep it on.

**Exception:** React 19's `use()` **can** be called conditionally.

---

### 3.11 Stale Closures

**Explanation**
Each render creates new functions that **capture the props/state of that render**. If a function is created once (an effect with `[]`, a `setInterval` callback, an event listener) and runs later, it sees **old values**.

```jsx
// ❌ count always reads 0 → the counter gets stuck at 1
function Timer() {
  const [count, setCount] = useState(0);
  useEffect(() => {
    const id = setInterval(() => setCount(count + 1), 1000);
    return () => clearInterval(id);
  }, []);                                   // closure captured count = 0
  return <p>{count}</p>;
}

// ✅ Fix 1: functional update (doesn't need count)
setInterval(() => setCount(c => c + 1), 1000);

// ✅ Fix 2: add to deps (re-creates the interval each change)
useEffect(() => { ... }, [count]);

// ✅ Fix 3: latest-value ref (for callbacks that must see fresh values)
function useLatest(value) {
  const ref = useRef(value);
  useLayoutEffect(() => { ref.current = value; });
  return ref;
}
```

**Scenario:** *"A WebSocket `onmessage` handler always uses the initial filter value."* The handler was registered once in a mount-only effect and closed over the first `filter`. Either re-subscribe when `filter` changes (add it to the deps), or read the filter from a ref inside the handler. (React 19.2's `useEffectEvent` is built for exactly this.)

**Tip:** Don't silence the `exhaustive-deps` lint rule. It's usually pointing at a stale closure.

---

### 3.12 Custom Hooks

**Explanation**
A custom hook is a **function starting with `use`** that calls other hooks. It shares **stateful logic**, not state: every component that calls it gets its own copy of the state.

Design guidelines:
- Name it after what it does (`useOnlineStatus`, not `useEffectForOnline`).
- Return a tuple `[value, setter]` for simple hooks, an object `{ data, error, isLoading }` for more.
- Keep returned functions stable (`useCallback`) when people will put them in dependency arrays.
- Always clean up subscriptions and timers.

```jsx
// useDebounce
function useDebounce(value, delay = 300) {
  const [debounced, setDebounced] = useState(value);
  useEffect(() => {
    const t = setTimeout(() => setDebounced(value), delay);
    return () => clearTimeout(t);
  }, [value, delay]);
  return debounced;
}

// useLocalStorage
function useLocalStorage(key, initial) {
  const [value, setValue] = useState(() => {
    try { const s = localStorage.getItem(key); return s !== null ? JSON.parse(s) : initial; }
    catch { return initial; }
  });
  useEffect(() => { localStorage.setItem(key, JSON.stringify(value)); }, [key, value]);
  return [value, setValue];
}

// useFetch
function useFetch(url) {
  const [state, setState] = useState({ data: null, error: null, loading: true });
  const [reloadKey, setReloadKey] = useState(0);

  useEffect(() => {
    const ctrl = new AbortController();
    setState(s => ({ ...s, loading: true, error: null }));
    fetch(url, { signal: ctrl.signal })
      .then(r => { if (!r.ok) throw new Error(`HTTP ${r.status}`); return r.json(); })
      .then(data => setState({ data, error: null, loading: false }))
      .catch(error => { if (error.name !== 'AbortError') setState({ data: null, error, loading: false }); });
    return () => ctrl.abort();
  }, [url, reloadKey]);

  const refetch = useCallback(() => setReloadKey(k => k + 1), []);
  return { ...state, refetch };
}

// useClickOutside
function useClickOutside(ref, handler) {
  useEffect(() => {
    const listener = e => { if (ref.current && !ref.current.contains(e.target)) handler(e); };
    document.addEventListener('pointerdown', listener);
    return () => document.removeEventListener('pointerdown', listener);
  }, [ref, handler]);
}
```

**Scenario:** *"Two components use `useFetch('/api/user')`. Do they share data?"*
No. Each call has its own state and makes its own request. To share and de-duplicate, put the data in Context or use React Query, which caches by query key.

**Testing:** `renderHook` from `@testing-library/react`.

---

## 4. Class Components (Legacy)

### 4.1 Lifecycle Methods & Hook Equivalents

**Explanation**
Class components go through three phases: **Mounting → Updating → Unmounting**, plus error handling.

| Phase | Method | Hook equivalent |
|---|---|---|
| Mount | `constructor` | `useState` initializer |
| Mount | `componentDidMount` | `useEffect(fn, [])` |
| Update | `shouldComponentUpdate` | `React.memo` |
| Update | `getSnapshotBeforeUpdate` | *none* (rare: scroll position) |
| Update | `componentDidUpdate(prevProps, prevState)` | `useEffect(fn, [deps])` |
| Unmount | `componentWillUnmount` | cleanup from `useEffect` |
| Both | `static getDerivedStateFromProps` | compute during render / `key` reset |
| Error | `getDerivedStateFromError`, `componentDidCatch` | **none**, still class-only |

```jsx
class UserProfile extends React.Component {
  state = { user: null };

  componentDidMount() { this.load(); }

  componentDidUpdate(prevProps) {
    if (prevProps.userId !== this.props.userId) this.load();   // compare manually
  }

  componentWillUnmount() { this.controller?.abort(); }

  load() {
    this.controller?.abort();
    this.controller = new AbortController();
    fetch(`/api/users/${this.props.userId}`, { signal: this.controller.signal })
      .then(r => r.json())
      .then(user => this.setState({ user }))
      .catch(() => {});
  }

  render() { return <div>{this.state.user?.name}</div>; }
}

// Hook version: one effect replaces three lifecycle methods
function UserProfile({ userId }) {
  const [user, setUser] = useState(null);
  useEffect(() => {
    const ctrl = new AbortController();
    fetch(`/api/users/${userId}`, { signal: ctrl.signal }).then(r => r.json()).then(setUser).catch(() => {});
    return () => ctrl.abort();
  }, [userId]);
  return <div>{user?.name}</div>;
}
```

**Scenario:** *"Why did hooks replace lifecycles?"* In classes, related logic is split across `didMount`, `didUpdate` and `willUnmount`, while unrelated logic gets mixed together in one method. Hooks group code **by concern**, not by lifecycle stage, and custom hooks make reuse easy without HOC "wrapper hell".

**Removed/unsafe:** `componentWillMount`, `componentWillReceiveProps`, `componentWillUpdate` are now `UNSAFE_`-prefixed because they break with async rendering.

---

### 4.2 `setState` Behaviour

**Explanation**
- `this.setState(partial)` **merges** shallowly with the current state (unlike `useState`, which **replaces** it).
- It's batched and asynchronous. Use the **updater form** when the new state depends on the previous one.
- The second argument is a callback that runs after the update is applied.

```jsx
this.setState({ count: this.state.count + 1 });
this.setState({ count: this.state.count + 1 });    // +1 total

this.setState(prev => ({ count: prev.count + 1 }));
this.setState(prev => ({ count: prev.count + 1 })); // +2 total

this.setState({ saved: true }, () => console.log('updated:', this.state.saved));
```

**Gotcha when converting to hooks:**
```jsx
const [form, setForm] = useState({ name: '', email: '' });
setForm({ name: 'A' });                      // ❌ email is GONE (no merge)
setForm(f => ({ ...f, name: 'A' }));         // ✅
```

**`this` binding:**
```jsx
class Btn extends React.Component {
  handleClick = () => this.setState({ clicked: true });   // arrow class field: auto-bound
  render() { return <button onClick={this.handleClick}>Go</button>; }
}
```

---

### 4.3 `PureComponent` & `shouldComponentUpdate`

**Explanation**
- `shouldComponentUpdate(nextProps, nextState)` returns `false` to skip a render.
- `React.PureComponent` implements it with a **shallow comparison** of props and state.
- The function-component equivalent is `React.memo(Component, arePropsEqual?)`.

```jsx
class Row extends React.PureComponent {
  render() { return <li>{this.props.item.name}</li>; }
}

// Pitfall: inline objects/functions defeat shallow comparison
<Row item={item} style={{ color: 'red' }} onClick={() => select(item.id)} />   // re-renders every time
```

**Scenario:** *"PureComponent didn't help, it still re-renders."* The parent passes new object or function references each render. Hoist the constants out, and bind handlers once.

---

### 4.4 Error Boundaries (still class-only)

See [Section 13.1](#131-error-boundaries) for full details. In short: a class with `static getDerivedStateFromError` (to render a fallback) and/or `componentDidCatch` (to log). There's no hook equivalent; use the `react-error-boundary` package to wrap one.

---

## 5. Component Patterns

### 5.1 Higher-Order Components (HOC)

**Explanation**
A HOC is a **function that takes a component and returns a new component** with extra behaviour. It was popular before hooks (Redux `connect`, `withRouter`).

```jsx
function withAuth(Component) {
  function WithAuth(props) {
    const { user, loading } = useAuth();
    if (loading) return <Spinner />;
    if (!user) return <Navigate to="/login" />;
    return <Component {...props} user={user} />;
  }
  WithAuth.displayName = `withAuth(${Component.displayName || Component.name})`;
  return WithAuth;
}

const Dashboard = withAuth(DashboardPage);
```

**Drawbacks:**
- "Wrapper hell" in DevTools.
- **Prop collisions** (two HOCs both inject `data`).
- Hard to see where a prop comes from.
- Static methods and refs aren't forwarded automatically.

**Scenario:** *"When would you still use a HOC today?"* For cross-cutting concerns applied to *many* components the same way, especially route guards, feature-flag gates, or wrapping third-party components. For sharing logic, prefer custom hooks.

---

### 5.2 Render Props

**Explanation**
A component takes a **function as a prop** (often `children`) and calls it with its internal state, so the **caller decides what to render**.

```jsx
function MouseTracker({ children }) {
  const [pos, setPos] = useState({ x: 0, y: 0 });
  return (
    <div onMouseMove={e => setPos({ x: e.clientX, y: e.clientY })} style={{ height: 300 }}>
      {children(pos)}
    </div>
  );
}

<MouseTracker>
  {({ x, y }) => <p>Mouse at {x}, {y}</p>}
</MouseTracker>
```

Today most render-prop components are replaced by a hook (`useMousePosition()`), but the pattern lives on in libraries: `<Formik>{formik => ...}</Formik>`, react-window's row renderer, Downshift.

**Scenario:** *"Hook vs render prop?"* A hook is simpler. A render prop is still useful when the logic needs a **DOM wrapper** (like the `div` with `onMouseMove` above), or when you want to control *where* in the tree the rendering happens.

---

### 5.3 Compound Components

**Explanation**
A group of components that **work together and share state through Context**, giving a flexible, declarative API. Examples: `<select>`/`<option>`, Radix UI, Headless UI, Reach.

```jsx
const TabsCtx = createContext(null);

function Tabs({ defaultValue, children }) {
  const [active, setActive] = useState(defaultValue);
  return <TabsCtx.Provider value={{ active, setActive }}>{children}</TabsCtx.Provider>;
}

Tabs.List = function TabList({ children }) {
  return <div role="tablist">{children}</div>;
};

Tabs.Tab = function Tab({ value, children }) {
  const { active, setActive } = useContext(TabsCtx);
  return (
    <button role="tab" aria-selected={active === value} onClick={() => setActive(value)}>
      {children}
    </button>
  );
};

Tabs.Panel = function TabPanel({ value, children }) {
  const { active } = useContext(TabsCtx);
  return active === value ? <div role="tabpanel">{children}</div> : null;
};

// Usage: consumer controls structure & order
<Tabs defaultValue="profile">
  <Tabs.List>
    <Tabs.Tab value="profile">Profile</Tabs.Tab>
    <Tabs.Tab value="billing">Billing</Tabs.Tab>
  </Tabs.List>
  <Tabs.Panel value="profile"><Profile /></Tabs.Panel>
  <Tabs.Panel value="billing"><Billing /></Tabs.Panel>
</Tabs>
```

**Scenario:** *"Your `<Tabs tabs={[...]} />` component has 15 props for icons, badges, disabled states, custom layouts…"*
That's "prop explosion". Refactor to compound components so consumers put the pieces together themselves, and the component only owns the shared state and accessibility behaviour.

---

### 5.4 Container vs Presentational

**Explanation**
- **Presentational**: how things *look*. Get data via props, no data fetching, easy to test and use in Storybook.
- **Container**: how things *work*. Fetch data, hold state, pass props down.

With hooks, the "container" part is usually a **custom hook**, but splitting logic from UI is still useful.

```jsx
// Presentational
function UserCard({ user, onFollow }) {
  return <div><h3>{user.name}</h3><button onClick={onFollow}>Follow</button></div>;
}

// Container (or a hook)
function UserCardContainer({ id }) {
  const { data: user, isLoading } = useUser(id);
  const follow = useFollow(id);
  if (isLoading) return <Skeleton />;
  return <UserCard user={user} onFollow={follow.mutate} />;
}
```

**Scenario:** *"Design system team wants components usable in Storybook without a backend."* Keep the design-system components presentational. Feature code adds the data around them.

---

### 5.5 Controlled Component Pattern & Prop Getters

**Explanation**
Reusable components often support **both controlled and uncontrolled** use, like native inputs do: if the parent passes `value`, it's controlled; otherwise the component manages its own state.

```jsx
function useControllableState({ value, defaultValue, onChange }) {
  const [internal, setInternal] = useState(defaultValue);
  const isControlled = value !== undefined;
  const current = isControlled ? value : internal;
  const setValue = useCallback(next => {
    if (!isControlled) setInternal(next);
    onChange?.(next);
  }, [isControlled, onChange]);
  return [current, setValue];
}

function Toggle({ on, defaultOn = false, onChange }) {
  const [isOn, setIsOn] = useControllableState({ value: on, defaultValue: defaultOn, onChange });
  return <button aria-pressed={isOn} onClick={() => setIsOn(!isOn)}>{isOn ? 'ON' : 'OFF'}</button>;
}

<Toggle />                                   // uncontrolled
<Toggle on={enabled} onChange={setEnabled} /> // controlled
```

**Prop getters** (from Downshift / React Table) return props the consumer spreads onto their own elements, so the component's handlers and the consumer's handlers get combined:
```jsx
function useToggle() {
  const [on, setOn] = useState(false);
  const getTogglerProps = ({ onClick, ...rest } = {}) => ({
    'aria-pressed': on,
    onClick: e => { onClick?.(e); setOn(o => !o); },
    ...rest,
  });
  return { on, getTogglerProps };
}
// <button {...getTogglerProps({ onClick: track })}>Toggle</button>
```

---

### 5.6 Portals

**Explanation**
`createPortal(children, domNode)` renders children **into a different DOM node** (usually `document.body`) while keeping them **in the same place in the React tree**.

Use for: modals, tooltips, dropdowns, toasts. Anything that must get out of a parent's `overflow: hidden`, `z-index` or `transform` stacking context.

**Key fact:** events **bubble through the React tree**, not the DOM tree. A click inside a portal reaches React ancestors' `onClick`. Context also still works.

```jsx
import { createPortal } from 'react-dom';

function Modal({ open, onClose, children }) {
  useEffect(() => {
    if (!open) return;
    const onKey = e => e.key === 'Escape' && onClose();
    document.addEventListener('keydown', onKey);
    document.body.style.overflow = 'hidden';
    return () => { document.removeEventListener('keydown', onKey); document.body.style.overflow = ''; };
  }, [open, onClose]);

  if (!open) return null;
  return createPortal(
    <div className="backdrop" onClick={onClose}>
      <div role="dialog" aria-modal="true" onClick={e => e.stopPropagation()}>
        {children}
      </div>
    </div>,
    document.body
  );
}
```

**Scenario:** *"A dropdown inside a card with `overflow: hidden` gets cut off."* Render the dropdown menu through a portal and position it using the trigger's `getBoundingClientRect()` (or use Floating UI).

**Note:** the native `<dialog>` element plus the **Popover API** now handle top-layer rendering in modern browsers too.

---

### 5.7 Refs to Components: `forwardRef` (and React 19)

**Explanation**
By default you can't attach a `ref` to a function component. To expose an inner DOM node:

```jsx
// React 18
const Input = React.forwardRef(function Input(props, ref) {
  return <input ref={ref} className="input" {...props} />;
});

// React 19: ref is a regular prop; forwardRef no longer needed
function Input({ ref, ...props }) {
  return <input ref={ref} className="input" {...props} />;
}

// Parent
const ref = useRef(null);
<Input ref={ref} />;
ref.current.focus();
```

Combine with `useImperativeHandle` to expose a limited API ([3.9](#39-other-built-in-hooks)).

**Scenario:** *"A design-system `<Button>` breaks focus management in a modal library."* The library needs a ref to the DOM node but `Button` doesn't forward it. Forward the ref to the underlying `<button>`. **Every design-system primitive should forward refs.**

**Callback refs:** `ref={node => { ... }}` runs when the node is attached/detached. Useful for measuring dynamic elements. In React 19 a callback ref can return a cleanup function.

---

### 5.8 Composition vs Inheritance

**Explanation**
React recommends **composition** (props, `children`, slots) over class inheritance. The React team says they haven't found any use case where an inheritance hierarchy would be better.

```jsx
// Specialisation through composition
function Dialog({ title, children, footer }) {
  return (
    <div className="dialog">
      <h2>{title}</h2>
      <div className="body">{children}</div>
      <footer>{footer}</footer>
    </div>
  );
}

function ConfirmDialog({ onConfirm, onCancel, message }) {
  return (
    <Dialog title="Are you sure?" footer={<><button onClick={onCancel}>Cancel</button><button onClick={onConfirm}>OK</button></>}>
      {message}
    </Dialog>
  );
}
```

**Scenario:** *"Create `PrimaryButton`, `DangerButton`, `IconButton` by extending `BaseButton`?"* No. Make one `Button` with a `variant` prop, plus composition for icons (`<Button icon={<Trash />}>`).

---

## 6. State Management

### 6.1 Types of State: Local, Global, Server, URL, Form

**Explanation**
Most "state management" problems come from treating every kind of state the same way. Sort it first:

| Type | Example | Best tool |
|---|---|---|
| **Local UI** | modal open, input value, hover | `useState` / `useReducer` |
| **Shared client** | theme, auth user, cart, sidebar | Context, Zustand, Redux |
| **Server / remote** | users list, product details | **React Query / SWR / RTK Query** |
| **URL** | filters, page, selected tab, search | router search params |
| **Form** | field values, validation, dirty | React Hook Form |
| **Derived** | totals, filtered list | compute during render / `useMemo` |

**Scenario:** *"Our Redux store is 80% API responses with manual `loading` / `error` flags."*
That's server state handled as client state. Move it to React Query or RTK Query, which give you caching, de-duplication, background refetching and invalidation. Redux then shrinks to the real client state.

**Tip:** "Keep state as local as possible, and lift it only when needed." Also: **don't store derived data** in state.

---

### 6.2 Context API & Its Limits

**Explanation**
Context is a **way to pass values down (dependency injection), not a state manager**. It has no selectors, so **every consumer re-renders when the value changes**.

```jsx
// ❌ One big context: changing `cart` re-renders theme consumers too
<AppContext.Provider value={{ user, theme, cart, setCart }}>

// ✅ Split by domain & update frequency; separate state from actions
const CartStateCtx = createContext();
const CartActionsCtx = createContext();

function CartProvider({ children }) {
  const [cart, dispatch] = useReducer(cartReducer, []);
  return (
    <CartActionsCtx.Provider value={dispatch}>       {/* stable */}
      <CartStateCtx.Provider value={cart}>{children}</CartStateCtx.Provider>
    </CartActionsCtx.Provider>
  );
}
// "Add to cart" buttons use only CartActionsCtx → don't re-render when cart changes
```

**Scenario:** *"When is Context enough vs. needing Redux/Zustand?"*
Context is enough for values that **change rarely** and are read widely (theme, locale, auth). Use a store when state **changes often**, many components read **different slices** (you need selectors), or you need middleware, devtools or persistence.

---

### 6.3 Redux & Redux Toolkit

**Explanation**
Redux = **single store**, state changed only by **dispatching actions** to **pure reducers**. Predictable, can be debugged with time travel, has middleware.

**Redux Toolkit (RTK)** is the official, recommended way to write Redux:
- `configureStore`: sets up devtools and thunk middleware.
- `createSlice`: generates action creators and reducers, and uses **Immer** so you can write "mutating" code safely.
- `createAsyncThunk`: async flows.
- **RTK Query**: built-in data fetching and caching.

```jsx
// cartSlice.js
import { createSlice, createAsyncThunk } from '@reduxjs/toolkit';

export const fetchCart = createAsyncThunk('cart/fetch', async () => (await fetch('/api/cart')).json());

const cartSlice = createSlice({
  name: 'cart',
  initialState: { items: [], status: 'idle' },
  reducers: {
    addItem(state, action) {                  // Immer: looks mutating, is immutable
      const existing = state.items.find(i => i.id === action.payload.id);
      if (existing) existing.qty += 1;
      else state.items.push({ ...action.payload, qty: 1 });
    },
    removeItem(state, action) {
      state.items = state.items.filter(i => i.id !== action.payload);
    },
  },
  extraReducers: builder => {
    builder
      .addCase(fetchCart.pending, s => { s.status = 'loading'; })
      .addCase(fetchCart.fulfilled, (s, a) => { s.status = 'idle'; s.items = a.payload; });
  },
});
export const { addItem, removeItem } = cartSlice.actions;
export default cartSlice.reducer;

// store.js
export const store = configureStore({ reducer: { cart: cartReducer } });

// Component
function CartCount() {
  const count = useSelector(s => s.cart.items.reduce((n, i) => n + i.qty, 0));  // re-renders only when count changes
  return <span>{count}</span>;
}
function AddButton({ product }) {
  const dispatch = useDispatch();
  return <button onClick={() => dispatch(addItem(product))}>Add</button>;
}
```

**Selectors & memoisation:** `useSelector` re-renders when the returned value changes (`===`). Returning a new array or object every time (`s.items.filter(...)`) causes re-renders on *every* store update. Use `createSelector` (Reselect) to memoise.

```js
const selectVisible = createSelector([s => s.todos, s => s.filter], (todos, f) => todos.filter(t => f === 'all' || t.status === f));
```

**Middleware:** functions that sit between dispatch and the reducer (logging, analytics, async). Thunks let you dispatch functions; **sagas** (generators) and **observables** (RxJS) handle complex async flows.

**Scenario:** *"When would you pick Redux in 2026?"* Large apps with complex client state shared across many features, many developers who benefit from a strict structure, the need for time-travel debugging or undo/redo, or an existing Redux codebase. For simpler needs, Zustand is lighter.

---

### 6.4 Zustand, Jotai, Recoil, MobX

**Explanation**
| Library | Model | Pros | Cons |
|---|---|---|---|
| **Zustand** | Small hook-based store, selectors | ~1KB, no Provider, little boilerplate | Less structure for very large teams |
| **Jotai** | **Atoms** (bottom-up), derived atoms | Fine-grained re-renders, works well with Suspense | Many atoms get hard to trace |
| **Recoil** | Atoms + selectors (Meta) | Similar to Jotai | **Archived / unmaintained**; avoid for new code |
| **MobX** | Observable, mutable, automatic tracking | Very little code, precise updates | "Magic", different mental model from React |
| **Redux Toolkit** | Single store, actions | Devtools, middleware, ecosystem | More ceremony |

```jsx
// Zustand
import { create } from 'zustand';

const useCart = create(set => ({
  items: [],
  add: item => set(s => ({ items: [...s.items, item] })),
  clear: () => set({ items: [] }),
}));

function CartBadge() {
  const count = useCart(s => s.items.length);    // re-renders only when length changes
  return <span>{count}</span>;
}

// Jotai
const countAtom = atom(0);
const doubleAtom = atom(get => get(countAtom) * 2);
function Counter() {
  const [count, setCount] = useAtom(countAtom);
  const double = useAtomValue(doubleAtom);
}
```

**Scenario:** *"A startup asks which state library to choose for a medium-sized dashboard."*
React Query for server state plus Zustand for the small amount of global client state. URL params for filters. That covers ~95% of cases with very little code.

---

### 6.5 React Query / SWR (Server State)

**Explanation**
Server state is different from client state: **someone else owns it**, it can get **out of date**, it's shared across components, and it needs caching, retries and de-duplication.

React Query (TanStack Query) gives you:
- **Caching by query key**: `['user', id]`
- **De-duplication**: 10 components using the same key make 1 request
- **Stale-while-revalidate**: show cached data immediately, refetch in the background
- **Refetch** on window focus, reconnect, or an interval
- Automatic **retries** with backoff
- **Mutations** with invalidation and **optimistic updates**
- Pagination / infinite queries, prefetching, SSR hydration

Key concepts: `staleTime` (how long data counts as fresh, default 0) vs `gcTime` (how long unused data stays in the cache, default 5 min).

```jsx
function useTodos() {
  return useQuery({
    queryKey: ['todos'],
    queryFn: () => fetch('/api/todos').then(r => r.json()),
    staleTime: 30_000,
  });
}

function useAddTodo() {
  const qc = useQueryClient();
  return useMutation({
    mutationFn: newTodo => fetch('/api/todos', { method: 'POST', body: JSON.stringify(newTodo) }).then(r => r.json()),

    // Optimistic update
    onMutate: async newTodo => {
      await qc.cancelQueries({ queryKey: ['todos'] });
      const previous = qc.getQueryData(['todos']);
      qc.setQueryData(['todos'], old => [...old, { ...newTodo, id: 'temp' }]);
      return { previous };
    },
    onError: (_err, _vars, ctx) => qc.setQueryData(['todos'], ctx.previous),   // rollback
    onSettled: () => qc.invalidateQueries({ queryKey: ['todos'] }),              // resync
  });
}

function Todos() {
  const { data, isPending, error } = useTodos();
  const add = useAddTodo();
  if (isPending) return <Spinner />;
  if (error) return <p>Error: {error.message}</p>;
  return (
    <>
      <button onClick={() => add.mutate({ title: 'New' })} disabled={add.isPending}>Add</button>
      <ul>{data.map(t => <li key={t.id}>{t.title}</li>)}</ul>
    </>
  );
}
```

**Scenario:** *"A user likes a post. The heart should fill instantly, even on a slow network."*
Use an optimistic update: in `onMutate`, update the cache straight away; roll back in `onError`; invalidate in `onSettled` to sync with the server. (React 19's `useOptimistic` does the same for Actions.)

**SWR vs React Query:** SWR is smaller and simpler. React Query has richer mutations, devtools, infinite queries and more cache control.

---

### 6.6 Choosing the Right Tool: Decision Framework

**Explanation**
Ask these questions in order:
1. **Is it derived?** → compute it, don't store it.
2. **Does it come from the server?** → React Query / SWR / RTK Query.
3. **Should it survive a refresh / be shareable via link?** → URL params.
4. **Is it form input?** → React Hook Form (or local state).
5. **Is it used by one component (and its children)?** → `useState`, lifted as needed.
6. **Used app-wide, changes rarely?** → Context.
7. **Used app-wide, changes often, many readers of different slices?** → Zustand / Redux Toolkit / Jotai.

**Scenario:** *"Product filters on an e-commerce listing: where do they live?"*
In the **URL** (`?category=shoes&sort=price`): links can be shared and bookmarked, the back button works, and SEO benefits. Read them with `useSearchParams` and pass them into the React Query key: `['products', filters]`.

---

## 7. Performance Optimization

### 7.1 The Optimisation Mindset

**Explanation**
1. **Measure first**: React DevTools Profiler, Chrome Performance tab, Lighthouse, Web Vitals.
2. Find the **real bottleneck**: is it re-renders, a slow single render, network, bundle size, images, or layout thrashing?
3. Fix with the **simplest** technique: change the structure first (move state down, pass children), then memoise.
4. **Measure again** to confirm.

**Scenario:** *"The app feels slow. Where do you start?"*
First, which part is slow? **Initial load** points to bundle size, network, SSR and images (LCP). **Interactions** point to re-renders and long tasks (INP). **Scrolling** points to big lists and layout. Profile it, then pick the fix.

---

### 7.2 `React.memo`

**Explanation**
`React.memo(Component)` skips re-rendering when props are **shallowly equal** to the last render's props.

It works only if:
- The component re-renders **often** with the **same props**, and
- Rendering it is **expensive** enough to matter, and
- Props are **referentially stable** (primitives, memoised objects and functions).

```jsx
const Chart = React.memo(function Chart({ data, onPointClick }) {
  /* heavy SVG rendering */
});

function Dashboard() {
  const [tab, setTab] = useState('a');
  const data = useMemo(() => transform(raw), [raw]);              // stable
  const onPointClick = useCallback(p => console.log(p), []);      // stable
  return (
    <>
      <Tabs value={tab} onChange={setTab} />
      <Chart data={data} onPointClick={onPointClick} />   {/* skipped on tab change */}
    </>
  );
}

// Custom comparison (rare)
const Row = React.memo(RowImpl, (prev, next) => prev.item.version === next.item.version);
```

**Scenario:** *"React.memo isn't working, the component still re-renders."*
Check for: (1) inline objects, arrays or functions in props; (2) `children` passed as JSX (a new element every time); (3) the component uses Context that changed; (4) its own state changed. Use the Profiler's **"Why did this render?"** to find out which.

---

### 7.3 Avoiding Unnecessary Re-renders: Structural Techniques

**Explanation**
These are often better than memoisation:

```jsx
// 1. Move state down
// ❌ Typing re-renders the whole page
function Page() {
  const [q, setQ] = useState('');
  return <><input value={q} onChange={e => setQ(e.target.value)} /><HeavyGrid /></>;
}
// ✅
function SearchInput() { const [q, setQ] = useState(''); return <input value={q} onChange={e => setQ(e.target.value)} />; }
function Page() { return <><SearchInput /><HeavyGrid /></>; }

// 2. Lift content up: pass expensive subtree as children
function ColorPicker({ children }) {
  const [color, setColor] = useState('red');
  return <div style={{ color }}><input type="color" onChange={e => setColor(e.target.value)} />{children}</div>;
}
<ColorPicker><HeavyGrid /></ColorPicker>   // HeavyGrid doesn't re-render on color change

// 3. Keep state close to where it's used, split contexts, and select narrow slices
const name = useStore(s => s.user.name);    // not useStore(s => s)
```

**Other quick wins:**
- Stable keys in lists.
- Don't define components inside components.
- Don't create new objects in render if they're passed to memoised children or used as deps.
- Debounce or throttle very frequent updates (scroll, resize, typing).

---

### 7.4 Code Splitting: `React.lazy`, `Suspense`, dynamic `import()`

**Explanation**
Ship less JavaScript up front. Split by **route** first, then by **heavy components** that aren't shown straight away (charts, editors, modals).

```jsx
import { lazy, Suspense } from 'react';

const Dashboard = lazy(() => import('./pages/Dashboard'));
const Settings  = lazy(() => import('./pages/Settings'));
const RichEditor = lazy(() => import('./components/RichEditor'));

function App() {
  return (
    <Suspense fallback={<PageSkeleton />}>
      <Routes>
        <Route path="/" element={<Dashboard />} />
        <Route path="/settings" element={<Settings />} />
      </Routes>
    </Suspense>
  );
}

// Prefetch on hover → instant navigation
const loadSettings = () => import('./pages/Settings');
<Link to="/settings" onMouseEnter={loadSettings}>Settings</Link>

// Lazy-load a heavy library only when needed
async function exportPdf() {
  const { jsPDF } = await import('jspdf');
  new jsPDF().text('Hello', 10, 10).save('a.pdf');
}
```

Notes:
- `lazy` needs a **default export** (or `.then(m => ({ default: m.Named }))`).
- Wrap lazy components in an **Error Boundary** too: chunk loading can fail after a deploy ("ChunkLoadError").
- Don't split too finely: too many tiny chunks means too many requests.

**Scenario:** *"After a deploy, some users get a blank screen when navigating."*
Old chunk files were deleted, and users who still have the old `index.js` request chunks that no longer exist. Fixes: keep old assets for a while after deploy, catch `ChunkLoadError` in an error boundary and reload the page once, or retry the dynamic import.

---

### 7.5 List Virtualization (Windowing)

**Explanation**
Rendering 10,000 rows creates 10,000+ DOM nodes, which makes the first render slow, uses a lot of memory and makes scrolling janky. **Virtualization renders only the visible rows plus a small buffer**, and adds spacer padding so the scrollbar still looks right.

Libraries: **TanStack Virtual**, react-window, react-virtuoso (supports dynamic heights).

```jsx
import { useVirtualizer } from '@tanstack/react-virtual';

function BigList({ rows }) {
  const parentRef = useRef(null);
  const virtualizer = useVirtualizer({
    count: rows.length,
    getScrollElement: () => parentRef.current,
    estimateSize: () => 40,
    overscan: 5,
  });

  return (
    <div ref={parentRef} style={{ height: 500, overflow: 'auto' }}>
      <div style={{ height: virtualizer.getTotalSize(), position: 'relative' }}>
        {virtualizer.getVirtualItems().map(v => (
          <div key={rows[v.index].id}
               style={{ position: 'absolute', top: 0, transform: `translateY(${v.start}px)`, height: v.size, width: '100%' }}>
            {rows[v.index].name}
          </div>
        ))}
      </div>
    </div>
  );
}
```

**Trade-offs:** Ctrl+F browser search only finds visible rows; screen readers can't see off-screen rows; dynamic row heights need measuring; SEO suffers.

**Scenario:** *"Chat app with 50,000 messages, scrolling is laggy."* Virtualize with dynamic height measurement and reverse (bottom-anchored) scrolling, and load older messages in pages as the user scrolls up. Alternative for simpler cases: the CSS `content-visibility: auto`.

---

### 7.6 Debouncing & Throttling in React

**Explanation**
- **Debounce**: run *after* the user stops for N ms (search input, autosave).
- **Throttle**: run at most once every N ms (scroll, resize, mouse move).

The common bug: creating the debounced function **inside render**, so each render gets a new debounce timer.

```jsx
// ❌ new debounced fn every render → doesn't debounce
const onChange = debounce(v => search(v), 300);

// ✅ stable debounced fn
function SearchBox({ onSearch }) {
  const onSearchRef = useRef(onSearch);
  useEffect(() => { onSearchRef.current = onSearch; });

  const debounced = useMemo(
    () => debounce(v => onSearchRef.current(v), 300),
    []
  );
  useEffect(() => () => debounced.cancel?.(), [debounced]);   // lodash debounce has cancel

  return <input onChange={e => debounced(e.target.value)} />;
}

// ✅ simplest: debounce the VALUE with a hook
const debouncedQuery = useDebounce(query, 300);
useEffect(() => { if (debouncedQuery) search(debouncedQuery); }, [debouncedQuery]);
```

**Scenario:** *"Search fires an API call on every keystroke and results flicker."* Debounce the query (300 ms), cancel or ignore stale requests (`AbortController`), and keep showing the previous results while loading (`placeholderData: keepPreviousData` in React Query).

---

### 7.7 Image & Asset Optimisation

**Explanation**
Images are usually the biggest part of a page's weight, and often its **LCP** element.
- Modern formats: **AVIF / WebP**.
- Responsive sizes: `srcset` + `sizes`.
- `loading="lazy"` for images below the fold. **Don't** lazy-load the LCP hero image; give it `fetchpriority="high"`.
- Always set `width`/`height` (or `aspect-ratio`) to avoid **CLS**.
- Use a CDN with automatic resizing; Next.js `<Image>` does this for you.
- Fonts: `font-display: swap`, preload key fonts, subset them.

```jsx
<img
  src="/hero-800.avif"
  srcSet="/hero-400.avif 400w, /hero-800.avif 800w, /hero-1600.avif 1600w"
  sizes="(max-width: 600px) 100vw, 800px"
  width={800} height={450}
  fetchPriority="high"
  alt="Product hero"
/>
<img src="/thumb.webp" loading="lazy" decoding="async" width={200} height={200} alt="" />
```

**Scenario:** *"Lighthouse says LCP is 4.5s on mobile."* Find the LCP element (usually the hero image). Then: preload it, serve it in the right size and format, make sure it isn't lazy-loaded, use SSR so it isn't waiting on JS, and use a CDN.

---

### 7.8 Profiling with React DevTools

**Explanation**
**React DevTools → Profiler tab:**
1. Click record, do the slow interaction, stop.
2. The **Flamegraph** shows each commit and how long each component took to render.
3. The **Ranked** view sorts components by render time.
4. Turn on **"Record why each component rendered"** to see "props changed: onClick", "hooks changed", "parent rendered".
5. **Components tab → "Highlight updates when components render"** shows re-renders visually as they happen.

**Chrome Performance tab:** long tasks (>50 ms), scripting vs rendering vs painting time, forced reflows.

**In code:**
```jsx
<Profiler id="Grid" onRender={(id, phase, actualDuration) => {
  if (actualDuration > 16) reportSlowRender({ id, phase, actualDuration });
}}>
  <Grid />
</Profiler>
```

**Scenario:** *"Describe how you fixed a performance problem."* (Prepare your own version.) "The Profiler showed every row of a 500-row table re-rendering when one cell was edited, because the parent passed `onChange={() => ...}` inline. We memoised the rows, made the callbacks stable with `useCallback`, and normalised the state so each row selected only its own data. Edit time went from 180 ms to 12 ms."

---

### 7.9 Web Vitals: LCP, INP, CLS

**Explanation**
| Metric | Measures | Good | Common React fixes |
|---|---|---|---|
| **LCP**: Largest Contentful Paint | Loading: when the main content appears | ≤ 2.5s | SSR/SSG, optimise hero image, smaller JS, preload, CDN |
| **INP**: Interaction to Next Paint (replaced FID in 2024) | Responsiveness of every interaction | ≤ 200ms | Fewer re-renders, `startTransition`, break up long tasks, virtualization, move work to Web Workers |
| **CLS**: Cumulative Layout Shift | Visual stability | ≤ 0.1 | Image dimensions, reserve space for ads and embeds, skeletons the same size as content, `font-display` |

Others: **TTFB** (server response time), **FCP** (first content appears).

```js
import { onLCP, onINP, onCLS } from 'web-vitals';
onLCP(sendToAnalytics); onINP(sendToAnalytics); onCLS(sendToAnalytics);
```

**Scenario:** *"INP is 600ms on a filter dropdown."* Selecting a filter re-renders a big list synchronously. Fixes: wrap the list update in `startTransition`, virtualize the list, memoise the rows, and move heavy filtering to `useMemo` or a Web Worker.

**Tip:** Lab data (Lighthouse) is not the same as field data (CrUX/RUM). Google ranks on **field data at the 75th percentile**.

---

### 7.10 Bundle Size & Tree Shaking

**Explanation**
- **Analyse**: `rollup-plugin-visualizer` (Vite), `webpack-bundle-analyzer`, `source-map-explorer`.
- **Tree shaking** removes unused exports. It needs **ES modules** and no side effects (`"sideEffects": false` in `package.json`).
- Common problems: `import _ from 'lodash'` (whole library) → `import debounce from 'lodash/debounce'` or `lodash-es`; `moment` → `date-fns` / `dayjs`; icon libraries imported all at once.
- Compression: Brotli/gzip; minify; modern browser targets (less polyfill code).

```js
// ❌ pulls everything
import { format } from 'date-fns/esm';  // fine, tree-shakeable
import * as Icons from 'lucide-react';  // ❌ may bundle all icons

// ✅
import { Search, X } from 'lucide-react';
```

**Scenario:** *"Main bundle is 2MB."* Run the analyser → split by route → lazy-load heavy components → swap heavy libraries → check tree shaking works → set a **bundle size budget in CI** (size-limit) so it doesn't grow again.

---

### 7.11 React Compiler

**Explanation**
The **React Compiler** (stable in the React 19 era) is a build-time Babel plugin that **memoises components and values automatically**. It looks at your code and adds the equivalent of `useMemo`, `useCallback` and `React.memo` where they're safe.

- It assumes your code **follows the Rules of React** (pure render, no mutation, hook rules).
- It skips components it can't safely optimise.
- Manual `useMemo`/`useCallback` become mostly unnecessary (they can still be used as escape hatches).

```js
// babel.config.js
module.exports = { plugins: ['babel-plugin-react-compiler'] };
```

**Scenario:** *"Should we remove all our useMemo/useCallback after adopting the compiler?"* Not in one big sweep. Leave them; they're harmless. New code can skip them. Use `eslint-plugin-react-compiler` / react-hooks lint to find code that breaks the rules and prevents optimisation.

---

## 8. Concurrent React (React 18+)

### 8.1 Concurrent Rendering: The Concept

**Explanation**
Before React 18, rendering was **synchronous and couldn't be interrupted**: once started, React blocked the main thread until it finished.

**Concurrent rendering** lets React:
- **Prepare several versions of the UI** at once.
- **Interrupt** low-priority rendering to handle urgent updates (typing, clicking).
- **Throw away** stale work.
- Show **already-visible content** instead of fallbacks while new content gets ready.

It's **opt-in per update**: you enable it with `createRoot` plus concurrent features (`startTransition`, `useDeferredValue`, Suspense). Urgent updates still render synchronously.

**Scenario:** *"Explain concurrent React to a junior developer."* "Think of React as a chef who can now pause chopping vegetables (a slow list render) to serve a customer at the counter (your keystroke), then go back to chopping, or throw the vegetables away if the order changed."

---

### 8.2 `useTransition` / `startTransition`

**Explanation**
Marks a state update as **non-urgent** (a transition). React keeps the UI responsive and can interrupt the transition render if something urgent comes in.

- `const [isPending, startTransition] = useTransition()` gives you a pending flag.
- `startTransition(fn)` (imported directly) has no pending flag; use it outside components.
- In React 19, transitions can be **async** (Actions).

```jsx
function SearchPage({ allItems }) {
  const [query, setQuery] = useState('');           // urgent: input stays responsive
  const [filter, setFilter] = useState('');         // non-urgent: heavy list
  const [isPending, startTransition] = useTransition();

  const handleChange = e => {
    setQuery(e.target.value);
    startTransition(() => setFilter(e.target.value));
  };

  const results = useMemo(() => allItems.filter(i => i.name.includes(filter)), [allItems, filter]);

  return (
    <>
      <input value={query} onChange={handleChange} />
      {isPending && <span>Updating…</span>}
      <div style={{ opacity: isPending ? 0.6 : 1 }}><List items={results} /></div>
    </>
  );
}

// Tab switching without showing a Suspense fallback again
const selectTab = next => startTransition(() => setTab(next));
```

**Rules:** you **can't** use transitions for controlled input values (the input must update urgently). Transitions don't debounce; they **deprioritise**.

**Scenario:** *"Switching tabs shows a full-page spinner because the new tab suspends."* Wrap the tab change in `startTransition`. React keeps showing the old tab (with `isPending`) until the new one is ready, instead of falling back to the Suspense spinner.

---

### 8.3 `useDeferredValue`

**Explanation**
Gives you a **deferred copy of a value**. React first re-renders with the old value, then re-renders in the background with the new one. Use it when you **don't control the state setter** (e.g. a value that arrives as a prop).

```jsx
function Results({ query }) {
  const deferredQuery = useDeferredValue(query);
  const isStale = query !== deferredQuery;

  return (
    <div style={{ opacity: isStale ? 0.5 : 1 }}>
      <SlowList query={deferredQuery} />    {/* must be memoised to benefit */}
    </div>
  );
}
const SlowList = React.memo(function SlowList({ query }) { /* heavy */ });
```

| | `useTransition` | `useDeferredValue` |
|---|---|---|
| Wraps | the **update** (setter call) | the **value** |
| Use when | you own the setState | value comes from props / a hook |
| Pending | `isPending` | compare `value !== deferred` |

**vs debounce:** debounce waits a fixed delay, even on fast machines. Deferred values render **as soon as possible** and can be interrupted, so they adapt to the device.

---

### 8.4 Suspense

**Explanation**
`<Suspense fallback={...}>` shows a fallback while something inside it is **not ready** ("suspends").

What can suspend:
- `React.lazy` components (code).
- Data via **Suspense-enabled libraries** (React Query `useSuspenseQuery`, Relay, Next.js RSC) or React 19's `use(promise)`.
- *Not* a plain `useEffect` fetch.

Patterns:
- **Nesting** boundaries for progressive loading (page shell → sidebar → content).
- Boundaries placed **where the user expects a loading state**.
- Pair with **Error Boundaries** for failures.

```jsx
function ProductPage({ id }) {
  return (
    <ErrorBoundary fallback={<ErrorView />}>
      <Suspense fallback={<HeaderSkeleton />}>
        <ProductHeader id={id} />
        <Suspense fallback={<ReviewsSkeleton />}>
          <Reviews id={id} />          {/* slow; doesn't block header */}
        </Suspense>
      </Suspense>
    </ErrorBoundary>
  );
}

function ProductHeader({ id }) {
  const { data } = useSuspenseQuery({ queryKey: ['product', id], queryFn: () => getProduct(id) });
  return <h1>{data.name}</h1>;     // no loading checks; Suspense handles it
}
```

**Scenario:** *"A page shows 5 separate spinners popping in at random times."* Group related content under one boundary so it appears together. Use nested boundaries only where loading progressively helps. Use transitions so content that's already visible doesn't get replaced by a fallback.

**Waterfall warning:** sibling components that suspend one after another can still create request waterfalls. Start the fetches early (prefetch in route loaders) or fetch in parallel.

---

### 8.5 Streaming SSR

**Explanation**
Traditional SSR is **all-or-nothing**: fetch all data → render the whole HTML → send it → download all JS → hydrate everything. The slowest part blocks the whole page.

React 18 **streaming SSR** (`renderToPipeableStream` / `renderToReadableStream`) with Suspense:
1. Sends the HTML shell and fallbacks **straight away**.
2. **Streams** each Suspense boundary's HTML as its data resolves (with inline scripts that put it in place).
3. **Selective hydration**: hydrates parts independently, and **prioritises the part the user is interacting with**.

```jsx
// server (Node)
import { renderToPipeableStream } from 'react-dom/server';
app.get('*', (req, res) => {
  const { pipe } = renderToPipeableStream(<App url={req.url} />, {
    bootstrapScripts: ['/client.js'],
    onShellReady() { res.setHeader('Content-Type', 'text/html'); pipe(res); },
    onError(err) { console.error(err); },
  });
});

// client
import { hydrateRoot } from 'react-dom/client';
hydrateRoot(document.getElementById('root'), <App />);
```

**Scenario:** *"Product page TTFB is 2s because of a slow recommendations API."* Wrap recommendations in `<Suspense>` and stream it. The main product HTML is sent immediately, and recommendations stream in when ready. In Next.js App Router, use `loading.tsx` or `<Suspense>` around async Server Components.

---

## 9. React 19

### 9.1 Actions, `useActionState`, `useFormStatus`

**Explanation**
**Actions** are async functions used inside transitions. React automatically tracks **pending state, errors, optimistic updates and form resets**.

- `<form action={fn}>`: React calls `fn(formData)` and resets uncontrolled fields when it succeeds.
- `useActionState(action, initialState)` → `[state, formAction, isPending]`.
- `useFormStatus()` (from `react-dom`): a child component reads the parent form's `pending` status without prop drilling.

```jsx
async function updateName(prevState, formData) {
  const name = formData.get('name');
  if (!name) return { error: 'Name is required' };
  try {
    await api.updateName(name);
    return { error: null, success: true };
  } catch (e) {
    return { error: e.message };
  }
}

function NameForm() {
  const [state, formAction, isPending] = useActionState(updateName, { error: null });
  return (
    <form action={formAction}>
      <input name="name" />
      <SubmitButton />
      {state.error && <p role="alert">{state.error}</p>}
      {state.success && <p>Saved!</p>}
    </form>
  );
}

function SubmitButton() {
  const { pending } = useFormStatus();    // reads nearest parent <form>
  return <button disabled={pending}>{pending ? 'Saving…' : 'Save'}</button>;
}
```

**Scenario:** *"How much boilerplate does this remove compared with React 18?"* No more `const [loading, setLoading]`, `try/finally setLoading(false)`, `e.preventDefault()`, a separate error state, or prop-drilling `isSubmitting` into the button. It also works with **progressive enhancement** (forms submit before JS loads when using Server Actions).

---

### 9.2 `useOptimistic`

**Explanation**
Shows an **optimistic version of state** while an async action is running. React **automatically goes back** to the real state when the action finishes (or fails).

```jsx
function Thread({ messages, sendMessage }) {
  const [optimisticMessages, addOptimistic] = useOptimistic(
    messages,
    (current, newText) => [...current, { id: crypto.randomUUID(), text: newText, sending: true }]
  );

  async function formAction(formData) {
    const text = formData.get('text');
    addOptimistic(text);              // shows instantly
    await sendMessage(text);          // when done, parent `messages` updates
  }

  return (
    <>
      {optimisticMessages.map(m => (
        <p key={m.id} style={{ opacity: m.sending ? 0.5 : 1 }}>{m.text}</p>
      ))}
      <form action={formAction}><input name="text" /><button>Send</button></form>
    </>
  );
}
```

**Scenario:** *"Chat message should appear instantly, greyed out until confirmed, and disappear with an error if it fails."* `useOptimistic` does exactly this. On failure, the optimistic entry disappears automatically; show a toast with a retry button.

---

### 9.3 The `use()` Hook

**Explanation**
`use(resource)` reads a **promise** or **context** during render.
- With a **promise**: the component **suspends** until it resolves; a rejection goes to the nearest Error Boundary.
- With **context**: like `useContext`, but **can be called conditionally** or inside loops (the only hook that can).
- The promise must be **cached/stable**. Create it in a Server Component, a loader or a cache, not in the same component's render (that would create a new promise every render and loop forever).

```jsx
// Server Component passes a promise to a Client Component
async function Page() {
  const commentsPromise = getComments();          // don't await → stream
  return (
    <Suspense fallback={<Spinner />}>
      <Comments commentsPromise={commentsPromise} />
    </Suspense>
  );
}

'use client';
function Comments({ commentsPromise }) {
  const comments = use(commentsPromise);
  return comments.map(c => <p key={c.id}>{c.text}</p>);
}

// Conditional context
function Heading({ show }) {
  if (!show) return null;
  const theme = use(ThemeContext);                // allowed
  return <h1 style={{ color: theme.color }}>Hi</h1>;
}
```

---

### 9.4 Other React 19 Changes

**Explanation**
- **`ref` as a prop**: no more `forwardRef` for function components.
- **`<Context>` as a provider**: `<ThemeContext value="dark">` instead of `.Provider`.
- **Ref cleanup functions**: a callback ref can return a cleanup.
- **Document metadata**: `<title>`, `<meta>`, `<link>` rendered anywhere get hoisted into `<head>`.
- **Stylesheets & scripts**: `<link rel="stylesheet" precedence="default">` with de-duplication and ordering; `preload`, `preinit` and `prefetchDNS` APIs.
- **Better hydration error messages** (shows a diff).
- **`useDeferredValue(value, initialValue)`**.
- **Server Components & Server Actions** are stable (used through frameworks).
- Removed: `propTypes` and `defaultProps` on function components (use TS / default parameters), string refs, legacy context, `ReactDOM.render` / `hydrate` (use `createRoot` / `hydrateRoot`).

```jsx
function BlogPost({ post }) {
  return (
    <article>
      <title>{post.title}</title>                    {/* hoisted to <head> */}
      <meta name="description" content={post.excerpt} />
      <h1>{post.title}</h1>
    </article>
  );
}

<div ref={node => {
  const observer = new ResizeObserver(() => {});
  observer.observe(node);
  return () => observer.disconnect();               // ref cleanup
}} />
```

**Scenario:** *"How would you plan an upgrade from 18 to 19?"* Upgrade to **18.3** first (it adds deprecation warnings). Fix the warnings: `defaultProps`, string refs, legacy context, `ReactDOM.render`. Run the official **codemods**. Upgrade the libraries (router, state, UI kit). Then upgrade React and run the tests and E2E. Finally, adopt new features one at a time.

---

### 9.5 Server Actions

**Explanation**
A **Server Action** is an async function marked `'use server'` that **runs on the server** but can be called from the client, as a form `action` or directly. The framework (Next.js) turns it into a POST endpoint for you.

```jsx
// app/actions.js
'use server';
import { revalidatePath } from 'next/cache';

export async function createTodo(formData) {
  const session = await auth();                         // ALWAYS authorise
  if (!session) throw new Error('Unauthorized');
  const title = String(formData.get('title') ?? '').trim();
  if (!title) return { error: 'Title required' };       // ALWAYS validate
  await db.todo.create({ data: { title, userId: session.user.id } });
  revalidatePath('/todos');
}

// app/todos/page.jsx (Server Component)
import { createTodo } from '../actions';
export default function Page() {
  return (
    <form action={createTodo}>
      <input name="title" />
      <button>Add</button>
    </form>
  );
}
```

**Security note:** Server Actions are **public HTTP endpoints**. Always check authentication, authorisation and input validation (Zod) inside them, never only in the UI.

**Scenario:** *"Do Server Actions replace the REST API?"* For your own web app's mutations, often yes. For mobile apps, third parties or a public API, you still need real API endpoints.

---

## 10. Rendering Strategies & SSR

### 10.1 CSR vs SSR vs SSG vs ISR

**Explanation**
| Strategy | HTML built | Pros | Cons | Use for |
|---|---|---|---|---|
| **CSR**: Client-Side Rendering | In the browser, after JS loads | Simple hosting (static), rich interactivity | Blank page until JS runs, weaker SEO, slow LCP on low-end devices | Dashboards, admin tools, apps behind login |
| **SSR**: Server-Side Rendering | On the server, **per request** | Fast first paint, SEO, fresh data | Server cost, TTFB depends on data, still needs hydration | Personalised pages, e-commerce, search results |
| **SSG**: Static Site Generation | At **build time** | Fastest (CDN), cheap | Data out of date until rebuild; long builds for huge sites | Blogs, docs, marketing |
| **ISR**: Incremental Static Regeneration | Static, then **regenerated in the background** after N seconds / on demand | SSG speed + fairly fresh data | Some users may see slightly old data | Product pages, news |

```jsx
// Next.js App Router equivalents
// SSG (default for static data)
export default async function Page() { const data = await fetch(url, { cache: 'force-cache' }); }

// ISR: revalidate every 60s
export const revalidate = 60;

// SSR: dynamic per request
export const dynamic = 'force-dynamic';
const data = await fetch(url, { cache: 'no-store' });

// CSR: a Client Component fetching in the browser
'use client';
function Widget() { const { data } = useQuery({ queryKey: ['w'], queryFn }); }
```

**Scenario:** *"Choose a rendering strategy for an e-commerce site."*
- Home/category pages: **ISR** (fast, mostly static).
- Product page: **ISR** for product content plus **CSR or streamed** price/stock (must be fresh).
- Cart/checkout: **SSR or CSR** (personal, no SEO).
- Account dashboard: **CSR**.
Mix them per route; there's no single right answer for the whole app.

---

### 10.2 React Server Components (RSC) vs Client Components

**Explanation**
**Server Components** run **only on the server** (at build time or per request). Their code is **never sent to the browser**; only their rendered output (the RSC payload) is.

| Server Component (default in Next App Router) | Client Component (`'use client'`) |
|---|---|
| Can be `async`, can `await` the DB / file system / secrets directly | Runs on the server (SSR) **and** in the client (hydration) |
| **Zero JS** shipped for the component | Its JS ships to the browser |
| ❌ No state, effects, event handlers, browser APIs | ✅ `useState`, `useEffect`, `onClick`, `window` |
| Can import Client Components | **Can't import** Server Components (but can receive them as `children`/props) |

```jsx
// app/products/[id]/page.jsx: Server Component
import { db } from '@/lib/db';
import AddToCart from './AddToCart';

export default async function ProductPage({ params }) {
  const product = await db.product.findUnique({ where: { id: params.id } });  // direct DB access
  return (
    <main>
      <h1>{product.name}</h1>
      <Markdown content={product.description} />   {/* heavy lib stays on server */}
      <AddToCart productId={product.id} />          {/* interactive island */}
    </main>
  );
}

// AddToCart.jsx: Client Component
'use client';
export default function AddToCart({ productId }) {
  const [qty, setQty] = useState(1);
  return <button onClick={() => addToCart(productId, qty)}>Add {qty}</button>;
}
```

**Key rules:**
- `'use client'` marks a **boundary**: everything that file imports becomes client code too.
- Props passed from Server → Client must be **serialisable** (no functions, except Server Actions; no class instances).
- Push `'use client'` **as far down the tree as possible**.
- Use the `server-only` package to stop server modules being imported into client code by mistake.

**Scenario:** *"Is RSC the same as SSR?"* No. **SSR** turns components into **HTML** for the first load, and then *all* their JS still ships and hydrates. **RSC** components **never hydrate or ship JS**, and can be refetched without a full page reload. They work **together**: RSC payload → SSR to HTML → hydrate only the Client Components.

---

### 10.3 Hydration & Hydration Mismatches

**Explanation**
**Hydration** = React on the client attaches event listeners and state to HTML that the **server already rendered**, instead of rebuilding the DOM. The client's first render must produce **the same output** as the server's.

**Common causes of mismatches:**
- `Date.now()`, `new Date().toLocaleString()` (time zones/locales differ), `Math.random()`
- `typeof window !== 'undefined'` branches in render
- Reading `localStorage` during render
- Invalid HTML nesting (`<div>` inside `<p>`, `<a>` inside `<a>`)
- Browser extensions changing the DOM

```jsx
// ❌ mismatch: server and client render different text
function Clock() { return <span>{new Date().toLocaleTimeString()}</span>; }

// ✅ render the same on both, then update after hydration
function Clock() {
  const [time, setTime] = useState(null);
  useEffect(() => {
    setTime(new Date().toLocaleTimeString());
    const id = setInterval(() => setTime(new Date().toLocaleTimeString()), 1000);
    return () => clearInterval(id);
  }, []);
  return <span>{time ?? '--:--'}</span>;
}

// ✅ client-only component (Next.js)
const Map = dynamic(() => import('./Map'), { ssr: false });

// ✅ known, unavoidable difference (e.g. timestamp)
<time suppressHydrationWarning>{new Date().toISOString()}</time>
```

**Scenario:** *"Dark mode flashes light for a moment on load (FOUC)."* The server doesn't know the user's theme, which is stored in localStorage. Fix: an inline blocking `<script>` in `<head>` that reads the theme and sets `class="dark"` on `<html>` before the first paint; or store the theme in a **cookie** so the server can render the right theme.

**Partial / progressive hydration:** islands architecture (Astro), selective hydration (React 18), and RSC all reduce hydration cost.

---

### 10.4 Next.js Essentials

**Explanation**
**App Router (`app/`) vs Pages Router (`pages/`)**

| | App Router (current) | Pages Router (legacy) |
|---|---|---|
| Default component | **Server Component** | Client (SSR'd) |
| Data fetching | `async` components + `fetch` | `getServerSideProps`, `getStaticProps` |
| Layouts | Nested `layout.tsx`, kept across navigations | `_app.tsx` only |
| Loading / error | `loading.tsx`, `error.tsx`, `not-found.tsx` files | Manual |
| Mutations | Server Actions | API routes |

```
app/
  layout.tsx          ← root layout (html/body)
  page.tsx            ← "/"
  dashboard/
    layout.tsx        ← shared sidebar, persists across child routes
    loading.tsx       ← Suspense fallback
    error.tsx         ← error boundary ('use client')
    page.tsx          ← "/dashboard"
    [id]/page.tsx     ← "/dashboard/123"
  api/users/route.ts  ← Route Handler (GET/POST)
middleware.ts         ← runs before requests (auth redirects, i18n, A/B)
```

**Caching layers** (often asked; the defaults changed between Next 14 and 15):
- Request memoisation (same `fetch` de-duplicated within one render)
- Data cache (`fetch` cache, `revalidate`, `revalidateTag`)
- Full route cache (static HTML/RSC)
- Router cache (client-side, for back/forward)

**Middleware example:**
```ts
// middleware.ts
export function middleware(req) {
  if (!req.cookies.get('session') && req.nextUrl.pathname.startsWith('/dashboard')) {
    return NextResponse.redirect(new URL('/login', req.url));
  }
}
export const config = { matcher: ['/dashboard/:path*'] };
```

**Scenario:** *"Updated product price isn't showing on the page."* Probably cached. Use `revalidateTag('products')` / `revalidatePath` after the update, set a suitable `revalidate`, or opt out with `cache: 'no-store'` for data that must always be live.

---

### 10.5 SEO in React Apps

**Explanation**
- CSR-only apps rely on crawlers running JS: slower and less reliable (and many social/link-preview bots don't run JS at all).
- Use **SSR/SSG** for pages that should be found by search.
- **Metadata**: `<title>`, meta description, canonical URL, Open Graph/Twitter cards (Next `generateMetadata`, React 19 `<title>`).
- **Structured data** (JSON-LD), `sitemap.xml`, `robots.txt`.
- Semantic HTML, real `<a href>` links (not `onClick` navigation), good Core Web Vitals.

```tsx
// Next.js
export async function generateMetadata({ params }) {
  const p = await getProduct(params.id);
  return {
    title: `${p.name} | Shop`,
    description: p.summary,
    openGraph: { images: [p.image] },
    alternates: { canonical: `/products/${p.slug}` },
  };
}
```

**Scenario:** *"Product links shared on WhatsApp/Slack show no preview."* Those bots don't run JS, so the OG tags must be in the **server-rendered HTML**.

---

## 11. Routing

### 11.1 React Router Basics: Nested & Dynamic Routes

**Explanation**
Client-side routing changes the URL with the **History API** without a full page reload, then renders the matching components.

```jsx
import { createBrowserRouter, RouterProvider, Outlet, Link, NavLink, useParams, useNavigate } from 'react-router-dom';

const router = createBrowserRouter([
  {
    path: '/',
    element: <RootLayout />,
    errorElement: <ErrorPage />,
    children: [
      { index: true, element: <Home /> },
      { path: 'users', element: <Users /> },
      { path: 'users/:userId', element: <UserDetail /> },     // dynamic segment
      { path: '*', element: <NotFound /> },
    ],
  },
]);

function RootLayout() {
  return (
    <>
      <nav>
        <NavLink to="/" end>Home</NavLink>
        <NavLink to="/users" className={({ isActive }) => (isActive ? 'active' : '')}>Users</NavLink>
      </nav>
      <Outlet />            {/* child route renders here */}
    </>
  );
}

function UserDetail() {
  const { userId } = useParams();
  const navigate = useNavigate();
  return <button onClick={() => navigate('/users')}>Back from {userId}</button>;
}

export default function App() { return <RouterProvider router={router} />; }
```

**Scenario:** *"Why use `<Link>` instead of `<a href>`?"* `<a>` triggers a full page reload, which loses app state and downloads everything again. `<Link>` uses `history.pushState` and renders on the client. It still renders a real `<a>`, so it's accessible and SEO-friendly.

---

### 11.2 Loaders & Actions (Data Routers)

**Explanation**
React Router 6.4+ (and v7 / Remix) let routes **load data before rendering** (`loader`) and **handle mutations** (`action`). This avoids "render → effect → fetch" waterfalls, because the data fetch starts **in parallel with code loading**, at navigation time.

```jsx
const router = createBrowserRouter([
  {
    path: 'users/:id',
    element: <User />,
    loader: async ({ params }) => {
      const res = await fetch(`/api/users/${params.id}`);
      if (!res.ok) throw new Response('Not found', { status: 404 });
      return res.json();
    },
    action: async ({ request, params }) => {
      const form = await request.formData();
      await fetch(`/api/users/${params.id}`, { method: 'PATCH', body: form });
      return redirect(`/users/${params.id}`);
    },
  },
]);

function User() {
  const user = useLoaderData();
  const navigation = useNavigation();          // 'idle' | 'loading' | 'submitting'
  return (
    <Form method="post">
      <input name="name" defaultValue={user.name} />
      <button disabled={navigation.state === 'submitting'}>Save</button>
    </Form>
  );
}
```

**Scenario:** *"The page loads in three steps: layout, then user, then user's posts."* That's a **fetch waterfall**. Move the fetching into route loaders so the requests run in parallel, or prefetch them (React Query `prefetchQuery` in the loader).

---

### 11.3 Protected / Private Routes

**Explanation**
Hide pages that need login, and redirect to login, keeping the intended destination so you can send the user back afterwards.

```jsx
function RequireAuth({ children, roles }) {
  const { user, loading } = useAuth();
  const location = useLocation();

  if (loading) return <FullPageSpinner />;
  if (!user) return <Navigate to="/login" replace state={{ from: location }} />;
  if (roles && !roles.includes(user.role)) return <Navigate to="/403" replace />;
  return children;
}

// Layout-route style
function ProtectedLayout() {
  const { user } = useAuth();
  return user ? <Outlet /> : <Navigate to="/login" replace />;
}

// after login
const from = location.state?.from?.pathname || '/';
navigate(from, { replace: true });
```

**Scenario (security):** *"Is route protection on the client enough?"* **No.** Client-side checks are only for **user experience**; anyone can change the JS bundle. The **API must enforce** authentication and authorisation on every request. In Next.js, also check in middleware or the server so protected HTML is never sent.

---

### 11.4 Lazy-Loaded Routes

**Explanation**
Route-level code splitting is the most effective split: each page loads its own JS.

```jsx
// React Router: route.lazy (loads component + loader in parallel)
const router = createBrowserRouter([
  { path: '/', element: <Home /> },
  { path: '/reports', lazy: () => import('./routes/reports') },   // exports Component, loader
]);

// or React.lazy + Suspense
const Reports = lazy(() => import('./routes/Reports'));
<Route path="/reports" element={<Suspense fallback={<Spinner />}><Reports /></Suspense>} />
```

**Scenario:** *"Admin pages are 40% of the bundle but only 2% of users are admins."* Lazy-load the `/admin/*` routes so regular users never download that code.

---

### 11.5 URL as State (Search Params)

**Explanation**
Put **shareable, bookmarkable UI state** in the URL: filters, sort, page, selected tab, search query. Benefits: links can be shared, the back/forward buttons work, the state survives a refresh, and search engines can index it.

```jsx
function ProductsPage() {
  const [params, setParams] = useSearchParams();
  const category = params.get('category') ?? 'all';
  const page = Number(params.get('page') ?? 1);

  const { data } = useQuery({
    queryKey: ['products', { category, page }],
    queryFn: () => fetchProducts({ category, page }),
    placeholderData: keepPreviousData,
  });

  const update = (key, value) =>
    setParams(prev => { const p = new URLSearchParams(prev); p.set(key, value); if (key !== 'page') p.set('page', '1'); return p; });

  return (
    <>
      <select value={category} onChange={e => update('category', e.target.value)}>
        <option value="all">All</option><option value="shoes">Shoes</option>
      </select>
      <ProductGrid items={data?.items} />
      <button onClick={() => update('page', String(page + 1))}>Next</button>
    </>
  );
}
```

**Scenario:** *"QA can't reproduce a bug because filters reset on refresh."* Move the filters from `useState` into search params. Now the URL alone reproduces the exact view.

---

## 12. Forms

### 12.1 Controlled Forms: The Basic Pattern

**Explanation**
One state object, one generic change handler, validation, and a submit handler.

```jsx
function SignupForm() {
  const [values, setValues] = useState({ email: '', password: '', agree: false });
  const [errors, setErrors] = useState({});
  const [touched, setTouched] = useState({});
  const [submitting, setSubmitting] = useState(false);

  const validate = v => {
    const e = {};
    if (!/^\S+@\S+\.\S+$/.test(v.email)) e.email = 'Enter a valid email';
    if (v.password.length < 8) e.password = 'Min 8 characters';
    if (!v.agree) e.agree = 'You must accept the terms';
    return e;
  };

  const handleChange = e => {
    const { name, value, type, checked } = e.target;
    setValues(v => ({ ...v, [name]: type === 'checkbox' ? checked : value }));
  };

  const handleBlur = e => setTouched(t => ({ ...t, [e.target.name]: true }));

  const handleSubmit = async e => {
    e.preventDefault();
    const errs = validate(values);
    setErrors(errs);
    setTouched({ email: true, password: true, agree: true });
    if (Object.keys(errs).length) return;
    setSubmitting(true);
    try { await api.signup(values); }
    catch (err) { setErrors({ form: err.message }); }
    finally { setSubmitting(false); }
  };

  return (
    <form onSubmit={handleSubmit} noValidate>
      <label htmlFor="email">Email</label>
      <input id="email" name="email" value={values.email} onChange={handleChange} onBlur={handleBlur}
             aria-invalid={!!errors.email} aria-describedby="email-err" />
      {touched.email && errors.email && <p id="email-err" role="alert">{errors.email}</p>}

      <label htmlFor="pw">Password</label>
      <input id="pw" type="password" name="password" value={values.password} onChange={handleChange} onBlur={handleBlur} />
      {touched.password && errors.password && <p role="alert">{errors.password}</p>}

      <label><input type="checkbox" name="agree" checked={values.agree} onChange={handleChange} /> I agree</label>

      {errors.form && <p role="alert">{errors.form}</p>}
      <button disabled={submitting}>{submitting ? 'Creating…' : 'Sign up'}</button>
    </form>
  );
}
```

**Scenario:** *"Users double-submit the payment form."* Disable the button while submitting, guard the handler (`if (submitting) return`), and make the **server idempotent** with an idempotency key. The UI alone isn't enough.

---

### 12.2 React Hook Form & Formik

**Explanation**
- **React Hook Form (RHF)**: **uncontrolled** by default (uses refs), so typing **doesn't re-render** the whole form. Small, fast, great TS support. The current industry default.
- **Formik**: controlled, one state object; more re-renders, and less actively maintained.

```jsx
import { useForm } from 'react-hook-form';
import { zodResolver } from '@hookform/resolvers/zod';
import { z } from 'zod';

const schema = z.object({
  email: z.string().email('Invalid email'),
  age: z.coerce.number().min(18, 'Must be 18+'),
  password: z.string().min(8),
  confirm: z.string(),
}).refine(d => d.password === d.confirm, { message: 'Passwords must match', path: ['confirm'] });

function Register() {
  const { register, handleSubmit, formState: { errors, isSubmitting } } = useForm({
    resolver: zodResolver(schema),
    mode: 'onBlur',
  });

  const onSubmit = async data => { await api.register(data); };

  return (
    <form onSubmit={handleSubmit(onSubmit)}>
      <input {...register('email')} placeholder="Email" />
      {errors.email && <p>{errors.email.message}</p>}
      <input {...register('age')} type="number" />
      {errors.age && <p>{errors.age.message}</p>}
      <input {...register('password')} type="password" />
      <input {...register('confirm')} type="password" />
      {errors.confirm && <p>{errors.confirm.message}</p>}
      <button disabled={isSubmitting}>Register</button>
    </form>
  );
}
```

For **controlled third-party inputs** (MUI, React Select, date pickers), use `<Controller control={control} name="x" render={({ field }) => <DatePicker {...field} />} />`.

---

### 12.3 Validation with Zod / Yup

**Explanation**
Schema-based validation gives you **one source of truth** for the rules, and **TypeScript types inferred** from the schema (Zod). The same schema can run on the **client and the server**.

```ts
import { z } from 'zod';

export const AddressSchema = z.object({
  street: z.string().min(1),
  pincode: z.string().regex(/^\d{6}$/, 'Invalid pincode'),
});
export const OrderSchema = z.object({
  items: z.array(z.object({ sku: z.string(), qty: z.number().int().positive() })).min(1),
  address: AddressSchema,
  coupon: z.string().optional(),
});
export type Order = z.infer<typeof OrderSchema>;   // TS type for free

// server-side (API route / server action)
const result = OrderSchema.safeParse(await req.json());
if (!result.success) return Response.json(result.error.flatten(), { status: 400 });
```

**Scenario:** *"Client validation passes but the server rejects the data. Rules got out of sync."* Share one Zod schema between the client and server (in a monorepo package).

---

### 12.4 Large & Dynamic Forms

**Explanation**
Problems: re-render cost, dynamic field arrays, multi-step wizards, auto-save, conditional fields.

```jsx
// Dynamic field arrays with RHF
function InvoiceItems() {
  const { control, register } = useFormContext();
  const { fields, append, remove } = useFieldArray({ control, name: 'items' });
  return (
    <>
      {fields.map((field, i) => (
        <div key={field.id}>                       {/* use field.id, NOT index */}
          <input {...register(`items.${i}.description`)} />
          <input type="number" {...register(`items.${i}.amount`, { valueAsNumber: true })} />
          <button type="button" onClick={() => remove(i)}>Remove</button>
        </div>
      ))}
      <button type="button" onClick={() => append({ description: '', amount: 0 })}>Add item</button>
    </>
  );
}

// Watch only what you need (re-renders only this component)
function Total() {
  const items = useWatch({ name: 'items' });
  const total = items?.reduce((s, i) => s + (Number(i.amount) || 0), 0) ?? 0;
  return <p>Total: {total}</p>;
}
```

**Strategies:**
- Uncontrolled inputs (RHF) / split into memoised sections.
- **Multi-step wizard**: one form state across steps (`FormProvider`), validate each step, keep progress in URL/session storage.
- **Auto-save**: debounce `watch` → save as a draft; show "Saved" / "Saving…"; handle conflicts.
- **Schema-driven forms**: render fields from JSON config for forms that admins configure.

**Scenario:** *"A 200-field insurance form lags on every keystroke."* It's probably controlled state at the top, so every keystroke re-renders 200 inputs. Move to RHF (uncontrolled), isolate computed fields with `useWatch`, split into steps or sections, and virtualize huge tables of inputs.

---

## 13. Error Handling & Resilience

### 13.1 Error Boundaries

**Explanation**
An Error Boundary is a **class component** that catches **errors thrown during rendering, in lifecycle methods and in constructors of its children**, and renders a fallback UI instead of unmounting the whole app (React unmounts the whole tree on an uncaught render error).

**It does NOT catch:**
- Errors in **event handlers** (use try/catch)
- **Async** code (`setTimeout`, promises) unless you re-throw the error into render
- **SSR** errors
- Errors **in the boundary itself**

```jsx
class ErrorBoundary extends React.Component {
  state = { error: null };

  static getDerivedStateFromError(error) {
    return { error };                                   // render fallback
  }

  componentDidCatch(error, info) {
    logToSentry(error, { componentStack: info.componentStack });   // side effects
  }

  reset = () => this.setState({ error: null });

  render() {
    if (this.state.error) {
      return this.props.fallback?.({ error: this.state.error, reset: this.reset }) ?? <p>Something went wrong.</p>;
    }
    return this.props.children;
  }
}

// Usage: granular boundaries
<Layout>
  <ErrorBoundary fallback={({ reset }) => <WidgetError onRetry={reset} />}>
    <RevenueChart />
  </ErrorBoundary>
  <ErrorBoundary fallback={() => <p>Feed unavailable</p>}>
    <ActivityFeed />
  </ErrorBoundary>
</Layout>
```

With the **`react-error-boundary`** library:
```jsx
import { ErrorBoundary, useErrorBoundary } from 'react-error-boundary';

<ErrorBoundary FallbackComponent={Fallback} onReset={() => qc.resetQueries()} resetKeys={[userId]}>
  <Profile />
</ErrorBoundary>

// Push async/event errors INTO the boundary
function Profile() {
  const { showBoundary } = useErrorBoundary();
  const onSave = async () => { try { await save(); } catch (e) { showBoundary(e); } };
}
```

**Scenario:** *"One broken widget crashes the whole dashboard."* Add an Error Boundary **per widget or route section**, so one failure shows a local fallback with a "Retry" button while the rest of the dashboard keeps working. Also have one **top-level** boundary as a last resort.

---

### 13.2 Handling Async & Event Handler Errors

**Explanation**
```jsx
// Event handler: try/catch + user feedback
async function handleDelete(id) {
  try {
    await api.delete(id);
    toast.success('Deleted');
  } catch (err) {
    toast.error(err.status === 403 ? 'You don’t have permission' : 'Delete failed, try again');
    logError(err);
  }
}

// Centralised fetch wrapper: normalise errors
class HttpError extends Error {
  constructor(status, body) { super(body?.message || `HTTP ${status}`); this.status = status; this.body = body; }
}
async function http(url, opts) {
  const res = await fetch(url, { ...opts, headers: { 'Content-Type': 'application/json', ...opts?.headers } });
  const body = res.headers.get('content-type')?.includes('json') ? await res.json() : null;
  if (!res.ok) throw new HttpError(res.status, body);
  return body;
}

// Global safety nets
window.addEventListener('unhandledrejection', e => logError(e.reason));
window.addEventListener('error', e => logError(e.error));
```

**Error UX levels:**
- **Field-level**: validation messages.
- **Component-level**: inline error + retry.
- **Page-level**: error boundary / `error.tsx`.
- **Global**: toast for background failures; offline banner.
- **401** → redirect to login / refresh the token; **403** → permission message; **5xx** → retry.

---

### 13.3 Fallback UI, Retries & Logging

**Explanation**
- **Retries with exponential backoff** for network errors. React Query retries 3 times by default.
- **Don't retry** 4xx errors (except 408/429).
- **Graceful degradation**: show cached/stale data with a "couldn't refresh" note instead of an error screen.
- **Logging/monitoring**: Sentry / Datadog RUM: errors with component stack, **source maps**, user/session context, release version, breadcrumbs.

```js
async function withRetry(fn, { retries = 3, base = 300 } = {}) {
  for (let attempt = 0; ; attempt++) {
    try { return await fn(); }
    catch (err) {
      const retriable = !err.status || err.status >= 500 || err.status === 429;
      if (!retriable || attempt >= retries) throw err;
      const delay = base * 2 ** attempt + Math.random() * 100;   // backoff + jitter
      await new Promise(r => setTimeout(r, delay));
    }
  }
}
```

**Scenario:** *"How do you know about frontend errors in production?"* Sentry with source maps uploaded in CI (not served publicly), alerts on error-rate spikes after a release, linked to the release version so you can roll back quickly, plus session replay for hard bugs.

---

## 14. Testing

### 14.1 Testing Philosophy: Pyramid vs Trophy

**Explanation**
- **Testing pyramid**: many unit tests, fewer integration tests, few E2E tests.
- **Testing trophy** (Kent C. Dodds), common for frontend: **static analysis** (TS, ESLint) → **unit** → **integration (the biggest layer)** → **E2E**.
- Integration tests (rendering a feature with real child components and a mocked network) give the **most confidence for the cost**.

**React Testing Library (RTL) principle:** *"The more your tests resemble the way your software is used, the more confidence they can give you."* Test **behaviour**, not implementation: no testing of internal state, hooks or class names.

**Query priority:** `getByRole` > `getByLabelText` > `getByPlaceholderText` > `getByText` > `getByDisplayValue` > `getByAltText` > `getByTitle` > `getByTestId` (last resort).

**Scenario:** *"Our tests break on every refactor even though the feature works."* They test implementation details (state values, component internals, snapshots). Rewrite them to test from the user's point of view with RTL queries.

---

### 14.2 Unit / Integration Testing with Jest/Vitest + RTL

**Explanation**
```jsx
// Counter.test.jsx
import { render, screen } from '@testing-library/react';
import userEvent from '@testing-library/user-event';
import Counter from './Counter';

test('increments when clicked', async () => {
  const user = userEvent.setup();
  render(<Counter />);

  expect(screen.getByRole('heading')).toHaveTextContent('Count: 0');
  await user.click(screen.getByRole('button', { name: /increment/i }));
  expect(screen.getByRole('heading')).toHaveTextContent('Count: 1');
});

// Form test
test('shows validation error for invalid email', async () => {
  const user = userEvent.setup();
  const onSubmit = vi.fn();                       // jest.fn() in Jest
  render(<LoginForm onSubmit={onSubmit} />);

  await user.type(screen.getByLabelText(/email/i), 'not-an-email');
  await user.click(screen.getByRole('button', { name: /log in/i }));

  expect(await screen.findByRole('alert')).toHaveTextContent(/valid email/i);
  expect(onSubmit).not.toHaveBeenCalled();
});
```

**`getBy` vs `queryBy` vs `findBy`:**
| | Not found | Async | Use for |
|---|---|---|---|
| `getBy*` | throws | no | element should be there |
| `queryBy*` | returns `null` | no | asserting **absence**: `expect(queryByText('x')).not.toBeInTheDocument()` |
| `findBy*` | rejects | **yes** (waits) | element appears after async work |

**`userEvent` vs `fireEvent`:** `userEvent` simulates real interactions (focus, keydown, input, keyup, click). Prefer it.

---

### 14.3 Mocking APIs with MSW

**Explanation**
**Mock Service Worker** intercepts requests at the **network level**, so your components use their real `fetch`/axios/React Query code. The same mocks work in tests, Storybook and local dev.

```js
// mocks/handlers.js
import { http, HttpResponse } from 'msw';
export const handlers = [
  http.get('/api/users', () => HttpResponse.json([{ id: 1, name: 'Asha' }])),
];

// setupTests.js
import { setupServer } from 'msw/node';
export const server = setupServer(...handlers);
beforeAll(() => server.listen());
afterEach(() => server.resetHandlers());
afterAll(() => server.close());

// UserList.test.jsx
test('renders users', async () => {
  renderWithProviders(<UserList />);
  expect(await screen.findByText('Asha')).toBeInTheDocument();
});

test('shows error on failure', async () => {
  server.use(http.get('/api/users', () => new HttpResponse(null, { status: 500 })));
  renderWithProviders(<UserList />);
  expect(await screen.findByRole('alert')).toHaveTextContent(/failed/i);
});
```

**Why not `jest.mock('axios')`?** That ties tests to the HTTP library you happen to use. If you switch from axios to fetch, every test breaks. MSW tests the real data flow.

---

### 14.4 Testing Hooks, Async & Context

**Explanation**
```jsx
// Custom hook
import { renderHook, act } from '@testing-library/react';
test('useCounter', () => {
  const { result } = renderHook(() => useCounter(5));
  act(() => result.current.increment());
  expect(result.current.count).toBe(6);
});

// Hook needing providers
const wrapper = ({ children }) => <QueryClientProvider client={new QueryClient()}>{children}</QueryClientProvider>;
const { result } = renderHook(() => useTodos(), { wrapper });
await waitFor(() => expect(result.current.isSuccess).toBe(true));

// Reusable render with all providers
function renderWithProviders(ui, { route = '/', user = null } = {}) {
  const qc = new QueryClient({ defaultOptions: { queries: { retry: false } } });
  return render(
    <QueryClientProvider client={qc}>
      <AuthContext.Provider value={{ user }}>
        <MemoryRouter initialEntries={[route]}>{ui}</MemoryRouter>
      </AuthContext.Provider>
    </QueryClientProvider>
  );
}

// Timers (debounce)
test('debounces search', async () => {
  vi.useFakeTimers();
  const user = userEvent.setup({ advanceTimers: vi.advanceTimersByTime });
  render(<Search onSearch={spy} />);
  await user.type(screen.getByRole('textbox'), 'abc');
  expect(spy).not.toHaveBeenCalled();
  vi.advanceTimersByTime(300);
  expect(spy).toHaveBeenCalledWith('abc');
  vi.useRealTimers();
});
```

**"not wrapped in act(...)" warning:** a state update happened after the test thought it was finished. Usually you're missing an `await findBy…` / `waitFor` for async work.

---

### 14.5 E2E Testing: Cypress / Playwright

**Explanation**
E2E tests run the **real app in a real browser**. Keep them for **critical user journeys**: login, checkout, signup, payments.

| | Playwright | Cypress |
|---|---|---|
| Browsers | Chromium, Firefox, WebKit | Chromium-family, Firefox (WebKit experimental) |
| Parallel | built-in, free | paid dashboard / plugins |
| Multi-tab / multi-origin | ✅ | limited |
| Speed | fast | good DX, time-travel UI |

```ts
// Playwright
import { test, expect } from '@playwright/test';

test('user can checkout', async ({ page }) => {
  await page.goto('/products/42');
  await page.getByRole('button', { name: 'Add to cart' }).click();
  await page.getByRole('link', { name: /cart/i }).click();
  await page.getByRole('button', { name: 'Checkout' }).click();
  await page.getByLabel('Card number').fill('4242 4242 4242 4242');
  await page.getByRole('button', { name: 'Pay' }).click();
  await expect(page.getByText('Order confirmed')).toBeVisible();
});
```

**Scenario:** *"E2E tests are flaky."* Causes: fixed `sleep`s, shared test data, timing assumptions, third-party services. Fixes: auto-waiting locators (role-based), isolated seeded data for each test, mocking third parties, retries plus traces to debug, and running in CI with the same viewport and settings every time.

---

### 14.6 Snapshot Testing & Visual Regression

**Explanation**
- **Snapshot tests** store the rendered output and fail when it changes.
  - ✅ Good for small, stable output (serialisers, error messages).
  - ❌ Large snapshots get approved without anyone reading them, and break on every markup change, so they protect very little.
- **Visual regression** (Chromatic, Percy, Playwright `toHaveScreenshot`) compares **pixels**. Better for design-system components.

```jsx
expect(render(<Badge status="error" />).container.firstChild).toMatchInlineSnapshot(`
  <span class="badge badge-error">Error</span>
`);
```

**Scenario:** *"PR has 40 snapshot files updated."* Reviewers can't check them in any meaningful way. Prefer explicit assertions plus visual regression tests for the UI.

---

## 15. Styling

### 15.1 Styling Approaches Compared

**Explanation**
| Approach | How | Pros | Cons |
|---|---|---|---|
| **Plain CSS / BEM** | global stylesheets | No tooling, fast | Global namespace collisions |
| **CSS Modules** | `Button.module.css` → scoped class names | Local scope, zero runtime, works with SSR | Dynamic styles need CSS variables |
| **Tailwind CSS** | utility classes | Fast to build with, consistent design tokens, tiny CSS output | Long className strings, learning curve |
| **styled-components / Emotion** | CSS-in-JS at **runtime** | Dynamic props, colocated, theming | Runtime cost, extra SSR setup, poor fit for RSC |
| **Zero-runtime CSS-in-JS** (vanilla-extract, Panda, Linaria, StyleX) | extracted at build time | Type-safe + no runtime | More build config |

```jsx
// CSS Modules
import styles from './Button.module.css';
<button className={`${styles.btn} ${primary ? styles.primary : ''}`}>Save</button>

// Tailwind + clsx for conditional classes
import clsx from 'clsx';
<button className={clsx('px-4 py-2 rounded font-medium', primary ? 'bg-blue-600 text-white' : 'bg-gray-100', disabled && 'opacity-50 cursor-not-allowed')}>
  Save
</button>

// styled-components
const Button = styled.button`
  padding: 8px 16px;
  background: ${p => (p.$primary ? p.theme.colors.primary : 'transparent')};
`;

// Dynamic values with CSS variables (works with any approach)
<div className={styles.bar} style={{ '--progress': `${pct}%` }} />
/* .bar::after { width: var(--progress); } */
```

**Scenario:** *"Moving to Next.js App Router; our styled-components setup is causing problems."* Runtime CSS-in-JS needs client-side context and doesn't work in Server Components. Migrate step by step to CSS Modules, Tailwind or a zero-runtime library.

---

### 15.2 CSS-in-JS Trade-offs

**Explanation**
- **Runtime cost**: styles are created and injected while rendering, which slows renders (most noticeable on large lists and low-end devices).
- **SSR**: you have to collect styles per request to avoid unstyled content flashing.
- **Bundle**: the library's runtime (~10–15KB).
- **React 18/19 concurrent rendering**: injecting styles during render conflicts with it (that's why `useInsertionEffect` exists).
- The React team and many large companies have moved towards **build-time** solutions.

**Tip:** "We liked colocated, themeable styles, so we kept the developer experience and dropped the runtime cost by moving to vanilla-extract / Tailwind."

---

### 15.3 Design Systems & Component Libraries

**Explanation**
A design system = **design tokens** (colours, spacing, typography, radii) + **components** + **guidelines** + **documentation** (Storybook).

Build vs buy:
- **Headless** (Radix UI, Headless UI, React Aria): behaviour and accessibility only; you add the styles. Most flexible.
- **Styled** (MUI, Chakra, Ant Design, Mantine): fast to start, harder to make look like your brand.
- **shadcn/ui**: copies Radix + Tailwind components **into your repo**, so you own the code.

```css
/* tokens.css: theming via CSS variables */
:root { --color-primary: #2563eb; --space-2: 8px; --radius: 6px; }
[data-theme='dark'] { --color-primary: #60a5fa; }
```

```jsx
// Variant API with cva (class-variance-authority)
const button = cva('inline-flex items-center rounded font-medium', {
  variants: {
    intent: { primary: 'bg-blue-600 text-white', secondary: 'bg-gray-100', danger: 'bg-red-600 text-white' },
    size: { sm: 'h-8 px-3 text-sm', md: 'h-10 px-4' },
  },
  defaultVariants: { intent: 'primary', size: 'md' },
});
function Button({ intent, size, className, ...props }) {
  return <button className={button({ intent, size, className })} {...props} />;
}
```

**Scenario:** *"Five teams each built their own Button and Modal."* Create a shared package with tokens, accessible primitives (built on Radix), Storybook docs, visual regression tests and semantic versioning. Help teams adopt it with codemods and lint rules that discourage raw `<button>` styling.

---

## 16. TypeScript with React

### 16.1 Typing Props, Children & Defaults

**Explanation**
```tsx
type ButtonProps = {
  label: string;
  variant?: 'primary' | 'secondary';      // union instead of string
  onClick?: (e: React.MouseEvent<HTMLButtonElement>) => void;
  children?: React.ReactNode;             // anything renderable
};

function Button({ label, variant = 'primary', onClick, children }: ButtonProps) {
  return <button className={variant} onClick={onClick}>{children ?? label}</button>;
}
```

**Children types:**
- `React.ReactNode`: anything renderable (string, number, element, array, null). **Use this for children.**
- `React.ReactElement`: a single JSX element only.
- `JSX.Element`: what JSX expressions return.

**`React.FC` debate:** it used to add `children` implicitly and caused problems with generics. Today the common style is to **type the props directly** (`function X(props: Props)`). `FC` is fine but not needed.

---

### 16.2 Typing State, Events & Refs

**Explanation**
```tsx
// State
const [user, setUser] = useState<User | null>(null);
const [items, setItems] = useState<Item[]>([]);           // empty array needs the annotation

// Events
const onChange = (e: React.ChangeEvent<HTMLInputElement>) => setQ(e.target.value);
const onSubmit = (e: React.FormEvent<HTMLFormElement>) => { e.preventDefault(); };
const onKey = (e: React.KeyboardEvent<HTMLInputElement>) => { if (e.key === 'Enter') submit(); };

// Refs
const inputRef = useRef<HTMLInputElement>(null);          // DOM ref
inputRef.current?.focus();
const timerRef = useRef<number | null>(null);             // mutable value

// useReducer with discriminated union actions
type Action =
  | { type: 'add'; item: Item }
  | { type: 'remove'; id: string }
  | { type: 'clear' };

function reducer(state: Item[], action: Action): Item[] {
  switch (action.type) {
    case 'add': return [...state, action.item];             // action.item is typed
    case 'remove': return state.filter(i => i.id !== action.id);
    case 'clear': return [];
  }
}
```

---

### 16.3 Extending Native Elements: `ComponentProps`, `PropsWithChildren`

**Explanation**
For wrapper components, inherit **all native attributes** so consumers can pass `type`, `aria-*`, `disabled`, etc.

```tsx
type ButtonProps = React.ComponentPropsWithoutRef<'button'> & {
  variant?: 'primary' | 'ghost';
  loading?: boolean;
};

function Button({ variant = 'primary', loading, children, disabled, ...rest }: ButtonProps) {
  return (
    <button {...rest} disabled={disabled || loading} data-variant={variant}>
      {loading ? <Spinner /> : children}
    </button>
  );
}

// Get props of another component
type ModalProps = React.ComponentProps<typeof Modal>;

// Children helper
type CardProps = React.PropsWithChildren<{ title: string }>;

// Utility types
type UserPreview = Pick<User, 'id' | 'name'>;
type UserUpdate = Partial<Omit<User, 'id'>>;
```

---

### 16.4 Generic Components

**Explanation**
Use generics when a component works with **any item type** but needs to keep it typed all the way through (lists, tables, selects).

```tsx
type ListProps<T> = {
  items: T[];
  getKey: (item: T) => string | number;
  renderItem: (item: T) => React.ReactNode;
};

function List<T>({ items, getKey, renderItem }: ListProps<T>) {
  return <ul>{items.map(i => <li key={getKey(i)}>{renderItem(i)}</li>)}</ul>;
}

// T inferred as User → user.name is autocompleted
<List items={users} getKey={u => u.id} renderItem={u => <b>{u.name}</b>} />

// Generic Select
type SelectProps<T> = { options: T[]; value: T; onChange: (v: T) => void; getLabel: (o: T) => string };
function Select<T>({ options, value, onChange, getLabel }: SelectProps<T>) { /* ... */ }

// In .tsx, arrow generics need a trailing comma or constraint: const f = <T,>(x: T) => x;
```

---

### 16.5 Discriminated Unions for Component State & Props

**Explanation**
Make **impossible states impossible to write**.

```tsx
// ❌ Booleans allow loading + error + data simultaneously
type Bad = { loading: boolean; error?: Error; data?: User };

// ✅
type FetchState<T> =
  | { status: 'idle' }
  | { status: 'loading' }
  | { status: 'success'; data: T }
  | { status: 'error'; error: Error };

function View({ state }: { state: FetchState<User> }) {
  switch (state.status) {
    case 'loading': return <Spinner />;
    case 'error':   return <p>{state.error.message}</p>;   // error is available only here
    case 'success': return <p>{state.data.name}</p>;       // data is available only here
    default:        return null;
  }
}

// Props that depend on each other
type LinkOrButton =
  | ({ as: 'link'; href: string } & React.ComponentPropsWithoutRef<'a'>)
  | ({ as: 'button'; onClick: () => void } & React.ComponentPropsWithoutRef<'button'>);
```

**Exhaustiveness check:**
```ts
default: { const _exhaustive: never = state; return _exhaustive; }   // compile error if a new status isn't handled
```

---

## 17. Security

### 17.1 XSS & `dangerouslySetInnerHTML`

**Explanation**
**XSS (Cross-Site Scripting)** = an attacker's script runs in your users' browsers and can steal tokens, act as the user, or change the page.

React **escapes all values in JSX by default**: `<div>{userInput}</div>` is safe, even if `userInput = "<script>…"`.

What's still dangerous in React:
1. `dangerouslySetInnerHTML` with untrusted HTML.
2. **`javascript:` URLs** in `href`/`src`.
3. Writing to the DOM directly via refs (`ref.current.innerHTML = …`).
4. Putting state into SSR HTML without escaping it (`<script>window.__STATE__ = ${JSON.stringify(state)}</script>`).
5. `eval`, `new Function`, third-party scripts.

```jsx
import DOMPurify from 'dompurify';

// ❌
<div dangerouslySetInnerHTML={{ __html: comment.html }} />

// ✅ sanitise (allowlist tags/attrs)
<div dangerouslySetInnerHTML={{ __html: DOMPurify.sanitize(comment.html, { ALLOWED_TAGS: ['b', 'i', 'a', 'p', 'ul', 'li'] }) }} />

// ✅ better: render markdown to React elements (react-markdown), no raw HTML at all
```

**Scenario:** *"We render CMS content with `dangerouslySetInnerHTML`."* Sanitise it with DOMPurify (on the server and/or client), restrict allowed tags, add a **CSP** as a second layer of defence, and limit who can edit the CMS.

---

### 17.2 `javascript:` URLs & Unsafe Attributes

**Explanation**
```jsx
// ❌ user profile website = "javascript:alert(document.cookie)"
<a href={user.website}>Website</a>

// ✅ validate the protocol
function safeUrl(url) {
  try {
    const u = new URL(url, window.location.origin);
    return ['http:', 'https:', 'mailto:'].includes(u.protocol) ? u.href : '#';
  } catch { return '#'; }
}
<a href={safeUrl(user.website)} target="_blank" rel="noopener noreferrer">Website</a>
```
- React 16.9+ **warns** about `javascript:` URLs, and newer versions block them, but don't rely on that alone.
- `target="_blank"` → add `rel="noopener noreferrer"` (prevents **reverse tabnabbing**; modern browsers set `noopener` by default).
- Never spread untrusted objects into props: `<div {...userControlledProps} />` could inject `dangerouslySetInnerHTML`.

---

### 17.3 Auth Token Storage: Cookies vs localStorage

**Explanation**
| | `localStorage` / `sessionStorage` | **httpOnly cookie** |
|---|---|---|
| JS can read it | ✅, so **any XSS can steal it** | ❌, not readable by JS |
| Sent automatically | No (you add the header) | Yes, so you **need CSRF protection** |
| Works across subdomains | No | Configurable |

**Recommended:** **httpOnly + Secure + SameSite=Lax/Strict** cookie for the session or refresh token. Optionally a short-lived access token **in memory** (a JS variable), refreshed silently. Or the **BFF (Backend-for-Frontend)** pattern: the browser holds only a session cookie, and the BFF talks to APIs with tokens.

```http
Set-Cookie: session=abc; HttpOnly; Secure; SameSite=Lax; Path=/; Max-Age=3600
```

**Scenario:** *"Where should we store the JWT?"* Not in localStorage, because one XSS bug would expose every user's token. Use an httpOnly cookie plus SameSite plus a CSRF token for state-changing requests; or keep it in memory with refresh-token rotation in an httpOnly cookie.

---

### 17.4 CSRF & CSP

**Explanation**
**CSRF (Cross-Site Request Forgery):** a malicious site makes the user's browser send a request to your site **with the user's cookies attached**.
Defences:
- `SameSite=Lax/Strict` cookies (the main defence today).
- **CSRF tokens** (synchroniser token or double-submit cookie) for state-changing requests.
- Check `Origin`/`Referer` headers.
- Never change state on `GET`.

**CSP (Content Security Policy):** an HTTP header that **limits where scripts, styles and other resources can load from**. A strong defence-in-depth layer against XSS.

```http
Content-Security-Policy:
  default-src 'self';
  script-src 'self' 'nonce-r4nd0m' https://js.stripe.com;
  style-src 'self' 'unsafe-inline';
  img-src 'self' data: https://cdn.example.com;
  connect-src 'self' https://api.example.com;
  frame-ancestors 'none';
```
- Use **nonces/hashes** instead of `'unsafe-inline'` for scripts.
- `frame-ancestors 'none'` prevents **clickjacking**.
- Roll it out with `Content-Security-Policy-Report-Only` first.

**Other security headers:** `Strict-Transport-Security`, `X-Content-Type-Options: nosniff`, `Referrer-Policy`, `Permissions-Policy`.

---

### 17.5 Other Frontend Security Topics

**Explanation**
- **Secrets in the bundle:** everything in client JS is public. `VITE_*` / `NEXT_PUBLIC_*` env vars get **embedded in the bundle**. Never put API secrets there.
- **Dependency security:** `npm audit`, Dependabot/Renovate, lockfiles, review new dependencies (supply-chain attacks), **Subresource Integrity** (SRI) for CDN scripts.
- **Client-side authorisation is only for UX:** hiding a button isn't security. The server must check every request.
- **Sensitive data:** don't log PII to the console/Sentry (scrub it); clear cached data on logout.
- **Open redirects:** validate `?redirect=` parameters against an allowlist.
- **postMessage:** always check `event.origin`.

```js
window.addEventListener('message', e => {
  if (e.origin !== 'https://trusted.example.com') return;
  handle(e.data);
});
```

---

## 18. Accessibility (a11y)

### 18.1 Semantic HTML First

**Explanation**
Native elements come with keyboard support, focus handling and screen-reader roles **for free**. The first rule of ARIA: **don't use ARIA if a native element does the job.**

```jsx
// ❌ Not focusable, no Enter/Space, no role
<div className="btn" onClick={save}>Save</div>

// ✅
<button type="button" onClick={save}>Save</button>

// ❌ Navigation via onClick
<span onClick={() => navigate('/about')}>About</span>
// ✅
<Link to="/about">About</Link>

// Landmarks & headings
<header>…</header>
<nav aria-label="Main">…</nav>
<main>
  <h1>Orders</h1>          {/* one h1, logical h2/h3 order */}
  <section aria-labelledby="recent"><h2 id="recent">Recent</h2></section>
</main>
<footer>…</footer>
```

**Images:** `alt="description"` for images that carry meaning; `alt=""` for decorative ones.
**Buttons vs links:** a link **goes somewhere** (changes the URL); a button **does something**.

---

### 18.2 ARIA Roles, States & Properties

**Explanation**
Use ARIA when you build **custom widgets** that HTML doesn't provide (tabs, combobox, tree, menu).

- **Roles**: `role="tab"`, `"dialog"`, `"alert"`, `"combobox"`, `"listbox"`
- **States**: `aria-expanded`, `aria-selected`, `aria-checked`, `aria-pressed`, `aria-disabled`, `aria-invalid`
- **Properties**: `aria-label`, `aria-labelledby`, `aria-describedby`, `aria-controls`, `aria-live`

```jsx
function Accordion({ id, title, open, onToggle, children }) {
  return (
    <div>
      <h3>
        <button aria-expanded={open} aria-controls={`${id}-panel`} id={`${id}-btn`} onClick={onToggle}>
          {title}
        </button>
      </h3>
      <div id={`${id}-panel`} role="region" aria-labelledby={`${id}-btn`} hidden={!open}>
        {children}
      </div>
    </div>
  );
}

// Icon-only button needs an accessible name
<button aria-label="Close dialog" onClick={onClose}><XIcon aria-hidden="true" /></button>
```

Follow the **WAI-ARIA Authoring Practices** patterns, or use **Radix / React Aria**, which implement them correctly.

---

### 18.3 Keyboard Navigation & Focus Management

**Explanation**
- Everything interactive must be **reachable with Tab** and usable with Enter/Space/arrow keys.
- **Visible focus**: never `outline: none` without a replacement; use `:focus-visible`.
- `tabIndex={0}` makes an element focusable in normal order; `tabIndex={-1}` makes it focusable only from code; **avoid positive values**.
- **Manage focus on route changes and dynamic content**: move focus to the new page's `<h1>` or to a modal.
- **Skip link**: "Skip to main content".
- **Roving tabindex** for composite widgets (tabs, toolbars, menus): one Tab stop, arrow keys move inside.

```jsx
// Focus heading on route change (SPAs don't do this by default)
function PageHeading({ children }) {
  const ref = useRef(null);
  const { pathname } = useLocation();
  useEffect(() => { ref.current?.focus(); }, [pathname]);
  return <h1 ref={ref} tabIndex={-1}>{children}</h1>;
}

// Skip link
<a href="#main" className="skip-link">Skip to main content</a>
<main id="main" tabIndex={-1}>…</main>

// Arrow key navigation in tabs
function onKeyDown(e, index) {
  if (e.key === 'ArrowRight') focusTab((index + 1) % tabs.length);
  if (e.key === 'ArrowLeft')  focusTab((index - 1 + tabs.length) % tabs.length);
  if (e.key === 'Home') focusTab(0);
  if (e.key === 'End')  focusTab(tabs.length - 1);
}
```

---

### 18.4 Accessible Modals, Forms & Live Regions

**Explanation**
**Modal checklist:**
1. `role="dialog"` + `aria-modal="true"` + `aria-labelledby` (its title).
2. **Move focus into** the modal when it opens (first field or close button).
3. **Trap focus** inside (Tab loops).
4. **Escape** closes it.
5. **Return focus** to the button that opened it on close.
6. Make the background inert (`inert` attribute) / lock scroll.
Or use native **`<dialog>` with `showModal()`**, which handles focus and inertness for you.

```jsx
function Dialog({ open, onClose, title, children }) {
  const ref = useRef(null);
  useEffect(() => {
    const d = ref.current;
    if (open && !d.open) d.showModal();          // native: focus trap + Esc + top layer
    if (!open && d.open) d.close();
  }, [open]);
  return (
    <dialog ref={ref} onClose={onClose} aria-labelledby="dlg-title">
      <h2 id="dlg-title">{title}</h2>
      {children}
      <button onClick={onClose}>Close</button>
    </dialog>
  );
}
```

**Forms:** every input has a `<label htmlFor>`; group related ones with `<fieldset>`/`<legend>`; link errors with `aria-describedby` and set `aria-invalid`; after a failed submit, **move focus to the first error** or an error summary; don't rely on colour alone.

**Live regions:** announce dynamic changes to screen readers.
```jsx
<div aria-live="polite" className="sr-only">{resultsCount} results found</div>
<div role="alert">{error}</div>   {/* assertive: announced immediately */}
```

**Scenario:** *"Screen reader users don't know the cart was updated after clicking Add."* Add a polite live region that announces "Added to cart, 3 items".

---

### 18.5 Testing Accessibility

**Explanation**
- **Linting**: `eslint-plugin-jsx-a11y`.
- **Automated**: axe-core (`jest-axe`, `@axe-core/playwright`), Lighthouse, Storybook a11y addon. These catch roughly **30–40%** of issues.
- **Manual**: keyboard-only walkthrough, screen readers (NVDA, VoiceOver, JAWS), 200% zoom, contrast checks (WCAG AA: 4.5:1 for normal text), reduced motion (`prefers-reduced-motion`).
- RTL's `getByRole` queries encourage accessible markup: if you can't find it by role, a screen reader probably can't either.

```jsx
import { axe } from 'jest-axe';
test('form has no a11y violations', async () => {
  const { container } = render(<SignupForm />);
  expect(await axe(container)).toHaveNoViolations();
});
```

**Standards:** WCAG 2.2 AA is the usual target, and legally required in many places (ADA in the US, the European Accessibility Act from 2025).

---

## 19. Architecture (Senior / Staff)

### 19.1 Folder Structure & Scaling a Codebase

**Explanation**
Organising **by file type** (`components/`, `hooks/`, `utils/`) works for small apps but turns into a mess at scale. Organise **by feature/domain** instead, and keep shared code separate.

```
src/
  app/                 # app shell: providers, router, global styles
  features/
    auth/
      api/             # queries/mutations for this feature
      components/
      hooks/
      types.ts
      index.ts         # PUBLIC API: only export what others may use
    cart/
    checkout/
  shared/
    ui/                # design-system primitives (Button, Modal)
    lib/               # http client, date utils
    hooks/
  pages/ or routes/    # thin: compose features
```

Rules that keep it scalable:
- **Features import from `shared`, never from other features' internals** (only through `index.ts`). Enforce this with ESLint `import/no-restricted-paths` / `eslint-plugin-boundaries`.
- **Colocate** tests, styles and stories next to the component.
- Pages/routes are thin and compose features.
- An established methodology: **Feature-Sliced Design**.

**Scenario:** *"Changing the cart broke the checkout and the profile pages."* The features depend on each other's internals. Define public APIs per feature, enforce import boundaries with lint, and move truly shared code into `shared/`.

---

### 19.2 Frontend System Design: The Framework

**Explanation**
Use the same structure for any frontend system design question (e.g. "Design a news feed / chat app / autocomplete / Google Docs / dashboard"). A common acronym is **RADIO**:

1. **R**equirements: functional (what it does) + non-functional (performance, scale, offline, a11y, i18n, devices). Ask questions to narrow the scope.
2. **A**rchitecture: high-level components, rendering strategy (CSR/SSR), state locations, server communication.
3. **D**ata model: entities, client-side store shape (normalised), what's cached.
4. **I**nterface (API): REST/GraphQL endpoints, pagination (cursor vs offset), real-time (WebSocket/SSE/polling), payload shape.
5. **O**ptimisations: performance, a11y, security, offline, error handling, observability, testing.

**Example: Design a News Feed (summary)**
- **Requirements**: infinite scroll, posts with images/video, like/comment, new posts arriving, works on 3G mobile.
- **Architecture**: SSR the first page for fast LCP → hydrate → CSR for later pages. Components: `Feed`, `Post`, `Composer`, `CommentList`.
- **Data**: normalised cache `{ posts: {byId}, users: {byId}, feed: [postIds] }` (React Query or a normalised store).
- **API**: `GET /feed?cursor=abc&limit=20` (**cursor pagination**: stable when new posts are inserted); `POST /posts/:id/like`; SSE/WebSocket for "N new posts" (shown as a banner, not inserted automatically, so the page doesn't jump).
- **Optimisations**: virtualized list; `IntersectionObserver` for loading more and for autoplay/pause of videos; lazy images with fixed aspect ratio (CLS); optimistic likes; prefetch the next page; skeletons; keep scroll position on back navigation; a11y (`role="feed"`, `aria-busy`, keyboard navigation); offline cache via service worker.

**Scenario follow-ups to prepare:** offset vs cursor pagination · WebSocket vs SSE vs polling · normalised vs nested cache · how to handle 1M items · how to keep a feed real-time without the page jumping.

---

### 19.3 Real-time Communication: WebSocket vs SSE vs Polling

**Explanation**
| | Short polling | Long polling | **SSE** | **WebSocket** |
|---|---|---|---|---|
| Direction | client → server | client → server | **server → client** | **two-way** |
| Protocol | HTTP | HTTP | HTTP (text/event-stream) | `ws://` upgrade |
| Reconnect | n/a | manual | **automatic** | manual |
| Use for | rare updates | legacy | notifications, live scores, **LLM streaming** | chat, multiplayer, collaborative editing |

```jsx
function useWebSocket(url, onMessage) {
  const handlerRef = useRef(onMessage);
  useEffect(() => { handlerRef.current = onMessage; });

  useEffect(() => {
    let ws, retry = 0, timer, closed = false;
    const connect = () => {
      ws = new WebSocket(url);
      ws.onopen = () => { retry = 0; };
      ws.onmessage = e => handlerRef.current(JSON.parse(e.data));
      ws.onclose = () => {
        if (closed) return;
        timer = setTimeout(connect, Math.min(1000 * 2 ** retry++, 30000));   // backoff reconnect
      };
    };
    connect();
    return () => { closed = true; clearTimeout(timer); ws?.close(); };
  }, [url]);
}

// SSE
useEffect(() => {
  const es = new EventSource('/api/notifications');
  es.onmessage = e => setNotifications(n => [JSON.parse(e.data), ...n]);
  return () => es.close();
}, []);
```

**Scenario:** *"Live stock prices for 500 tickers, UI is freezing."* Too many updates per second, each causing a render. Buffer the messages and apply them in batches (e.g. every 250 ms or with `requestAnimationFrame`), keep the data in an external store with **per-ticker selectors** so only changed rows re-render, and virtualize the list.

---

### 19.4 Micro-Frontends

**Explanation**
Split a large frontend into **independently built and deployed apps** owned by different teams (e.g. Search, Cart and Account teams).

**Integration approaches:**
- **Build-time** (npm packages): simple, but every change needs a re-release, so teams aren't really independent.
- **Run-time via Module Federation** (Webpack 5 / Rspack / Vite plugin): the host loads remote modules at run time and **shares dependencies** (one copy of React).
- **iframes**: strongest isolation, poor user experience (sizing, routing, a11y).
- **Web Components** as a framework-neutral wrapper.
- **Server-side composition** (edge includes).

```js
// host webpack.config.js
new ModuleFederationPlugin({
  name: 'shell',
  remotes: { cart: 'cart@https://cart.example.com/remoteEntry.js' },
  shared: { react: { singleton: true, requiredVersion: '^18.3.0' }, 'react-dom': { singleton: true } },
});

// host code
const CartWidget = lazy(() => import('cart/CartWidget'));
<ErrorBoundary fallback={<CartUnavailable />}><Suspense fallback={<Spinner />}><CartWidget /></Suspense></ErrorBoundary>
```

**Trade-offs:**
- ✅ Teams own and deploy their parts independently; incremental migration (e.g. Angular → React); fault isolation.
- ❌ Operational complexity, duplicated dependencies, inconsistent UX, shared-state and routing difficulty, performance cost, version mismatches.

**Scenario:** *"Should we adopt micro-frontends?"* Only if you have an **organisational** scaling problem (many teams blocked on each other's deploys). It isn't a technical performance fix. For a single team, a **modular monolith in a monorepo** is almost always better.

---

### 19.5 Monorepos (Nx, Turborepo)

**Explanation**
One repo, many packages/apps: `apps/web`, `apps/admin`, `packages/ui`, `packages/utils`, `packages/config`.

Benefits: **shared code without publishing**, atomic changes across packages, one set of tooling, easier refactoring.

Tools:
- **Package managers with workspaces**: pnpm (recommended), npm, yarn.
- **Turborepo**: task pipelines + **caching** (local and remote). Simple.
- **Nx**: caching + **affected-only** builds/tests (based on the dependency graph), generators, boundary rules. More features.

```json
// turbo.json
{
  "tasks": {
    "build": { "dependsOn": ["^build"], "outputs": ["dist/**", ".next/**"] },
    "test":  { "dependsOn": ["^build"] },
    "lint":  {}
  }
}
```

**Scenario:** *"CI takes 40 minutes in the monorepo."* Only build and test **affected** packages (`nx affected`, `turbo --filter=...[origin/main]`), turn on **remote caching**, run jobs in parallel, and split slow E2E suites into shards.

---

### 19.6 Build Tooling: Vite, Webpack, Babel/SWC

**Explanation**
| Tool | What it does |
|---|---|
| **Vite** | Dev: serves **native ES modules** + esbuild pre-bundling, so the dev server starts almost instantly and HMR is fast. Prod: Rollup (moving to **Rolldown**). Current default for SPAs. |
| **Webpack** | Mature bundler, huge plugin ecosystem, Module Federation. Slower dev experience. |
| **Rspack / Turbopack** | Rust-based, Webpack-compatible (Rspack) / Next.js default (Turbopack). |
| **Babel** | JS transpiler (JSX, new syntax → older), plugin-based (React Compiler is a Babel plugin). |
| **SWC / esbuild** | Rust/Go transpilers, 20–70× faster than Babel. |

Key concepts: **HMR / Fast Refresh** (keeps component state on edit), **tree shaking**, **code splitting**, **minification**, **source maps**, **content-hash filenames** for long-term caching, **env variables** at build time, **browserslist** targets.

**Scenario:** *"Migrating CRA (deprecated) to Vite."* Move `index.html` to the root and add the script module tag; `REACT_APP_*` → `VITE_*` (`import.meta.env`); adjust Jest → Vitest or keep Jest with babel-jest; fix SVG imports (`vite-plugin-svgr`); check proxy config; compare bundle output.

---

### 19.7 Migration Strategies

**Explanation**
Principles: **incremental, not a big-bang rewrite**; **tests before changes**; feature flags; measure.

- **Class → hooks**: only when you touch the file; tests first; use the lifecycle → hook mapping table.
- **JS → TS**: `allowJs`, convert one file at a time starting with leaf utilities and shared types, `strict` gradually (per-folder `tsconfig` or `// @ts-check`).
- **CRA → Vite / Next**: as above; for Next, adopt the App Router route by route.
- **React 17 → 18**: `createRoot`; check automatic batching assumptions; StrictMode double effects; update libraries.
- **React 18 → 19**: upgrade to 18.3, fix warnings, run codemods (`npx codemod react/19/...`), update types.
- **Redux (legacy) → RTK / React Query**: move server data to React Query slice by slice; convert remaining reducers to `createSlice`.
- **Legacy framework (Angular.js/jQuery) → React**: the **strangler fig** pattern: mount React components inside the old app (or the old app inside a React shell), and replace routes one at a time.

**Scenario:** *"Leadership wants a full rewrite in 6 months."* Push back with data: rewrites are risky (feature freeze, missed edge cases, slipping dates). Propose a strangler-fig migration: the highest-value or most-painful screens first, shipped continuously, with success metrics (bug rate, LCP, developer speed).

---

### 19.8 Internationalisation (i18n)

**Explanation**
- Libraries: **react-i18next**, **FormatJS / react-intl**, next-intl.
- Never join strings together; use **message keys with interpolation** and **ICU plural/select** rules.
- Format **dates, numbers and currency** with `Intl` APIs.
- **RTL** support (Arabic/Hebrew): `dir="rtl"`, CSS logical properties (`margin-inline-start`).
- **Lazy-load translation files** per locale and per namespace.
- Locale in the URL (`/fr/products`) for SEO, with `hreflang`.

```jsx
// en.json: { "cart": "{count, plural, =0 {Your cart is empty} one {# item} other {# items}}" }
const { t } = useTranslation();
<p>{t('cart', { count })}</p>

new Intl.NumberFormat('en-IN', { style: 'currency', currency: 'INR' }).format(123456.5); // ₹1,23,456.50
new Intl.DateTimeFormat('de-DE', { dateStyle: 'long' }).format(new Date());
new Intl.RelativeTimeFormat('en', { numeric: 'auto' }).format(-1, 'day');                // "yesterday"
```

**Scenario:** *"German text overflows buttons."* German is often 30% longer than English. Design for text expansion (no fixed widths), test with pseudo-localisation, and use CSS that lets text wrap.

---

### 19.9 Feature Flags, Observability & Frontend CI/CD

**Explanation**
**Feature flags** (LaunchDarkly, Unleash, GrowthBook, Statsig):
- Separate **deploy** from **release**; gradual rollouts (1% → 10% → 100%); instant kill switch; A/B tests.
- Evaluate flags on the server where you can, to avoid flicker; clean up old flags (they become tech debt).

```jsx
function Checkout() {
  const newFlow = useFlag('new-checkout');
  return newFlow ? <CheckoutV2 /> : <CheckoutV1 />;
}
```

**Observability:**
- **Errors**: Sentry (with source maps, release tags).
- **Performance / RUM**: Web Vitals by route, device and country (Datadog RUM, New Relic, SpeedCurve).
- **Product analytics**: funnels, events.
- **Session replay** (with PII masked).
- Alerts on error rate or Web Vitals getting worse after a release.

**Frontend CI/CD pipeline:**
```
PR → install (cached) → lint + typecheck → unit/integration tests → build
   → bundle-size check → preview deploy (Vercel/Netlify) → E2E on preview → visual regression
merge → build once → deploy to CDN (immutable hashed assets, index.html no-cache)
     → smoke tests → gradual rollout / flag → monitor → fast rollback
```

**Scenario:** *"A release broke checkout for Safari users."* Turn off the feature flag or roll back straight away (instant, since the assets are immutable). Then add a WebKit E2E job (Playwright) to CI, and set up alerts that break errors down by browser.

---

## 20. JavaScript Fundamentals

### 20.1 Closures, Scope & Hoisting

**Explanation**
- **Scope**: where variables can be accessed: global, function, block (`let`/`const`).
- **Closure**: a function **remembers the variables from the scope where it was defined**, even after that scope has finished running.
- **Hoisting**: declarations are moved to the top of their scope during compilation.
  - `var` → hoisted and initialised to `undefined`.
  - `let`/`const` → hoisted but **not initialised**. Accessing them before the declaration throws `ReferenceError` (the **Temporal Dead Zone**).
  - Function declarations → fully hoisted (can be called before their definition); function expressions/arrows → follow `var`/`let` rules.

```js
function makeCounter() {
  let count = 0;                  // private via closure
  return {
    inc: () => ++count,
    get: () => count,
  };
}
const c = makeCounter(); c.inc(); c.inc(); c.get(); // 2

console.log(a); // undefined (var hoisted)
var a = 1;
console.log(b); // ReferenceError (TDZ)
let b = 2;

hello();        // works
function hello() { console.log('hi'); }
```

**Classic interview question:**
```js
for (var i = 0; i < 3; i++) setTimeout(() => console.log(i), 0);  // 3 3 3 (one shared i)
for (let i = 0; i < 3; i++) setTimeout(() => console.log(i), 0);  // 0 1 2 (new i per iteration)
// var fix with IIFE closure:
for (var i = 0; i < 3; i++) ((j) => setTimeout(() => console.log(j), 0))(i);
```

**React connection:** hooks rely entirely on closures, and so do stale-closure bugs ([3.11](#311-stale-closures)).

---

### 20.2 `this`, `call`, `apply`, `bind`

**Explanation**
`this` is decided by **how a function is called**, not where it's defined (except for arrow functions).

| Call style | `this` |
|---|---|
| `obj.method()` | `obj` |
| `fn()` | `undefined` in strict mode / `globalThis` in sloppy mode |
| `new Fn()` | the new instance |
| `fn.call(x, a, b)` / `fn.apply(x, [a, b])` | `x` |
| `fn.bind(x)` | returns a new function with `this` permanently set to `x` |
| Arrow function | **lexical**: the `this` of the surrounding scope; can't be changed |

```js
const user = {
  name: 'Asha',
  greet() { return `Hi ${this.name}`; },
  greetLater() { setTimeout(() => console.log(this.name), 0); },   // arrow keeps `this`
};
const g = user.greet;
g();                       // "Hi undefined": lost `this`
g.call({ name: 'Ravi' });  // "Hi Ravi"
const bound = user.greet.bind(user); bound(); // "Hi Asha"

// bind polyfill (common question)
Function.prototype.myBind = function (ctx, ...preset) {
  const fn = this;
  return function (...args) { return fn.apply(ctx, [...preset, ...args]); };
};
```

**React connection:** class components needed `this.handleClick = this.handleClick.bind(this)` or arrow class fields, because `onClick={this.handleClick}` passes the method without its object.

---

### 20.3 The Event Loop: Microtasks vs Macrotasks

**Explanation**
JS runs on **one thread**. The event loop:
1. Runs all **synchronous code** (the call stack).
2. Then empties the **microtask queue** completely: Promise `.then/catch/finally`, `queueMicrotask`, `await` continuations, `MutationObserver`.
3. Then the browser may **render** (rAF callbacks run before paint).
4. Then takes **one macrotask**: `setTimeout`, `setInterval`, I/O, UI events, `MessageChannel`. Then back to step 2.

```js
console.log('1');
setTimeout(() => console.log('2'), 0);
Promise.resolve().then(() => console.log('3'));
(async () => {
  console.log('4');
  await null;
  console.log('5');
})();
queueMicrotask(() => console.log('6'));
console.log('7');
// Output: 1 4 7 3 5 6 2
```
Step by step: sync `1`, `4` (the async function runs synchronously until `await`), `7` → microtasks in the order they were queued: `3`, `5`, `6` → macrotask `2`.

**Harder one:**
```js
setTimeout(() => console.log('A'), 0);
Promise.resolve().then(() => {
  console.log('B');
  setTimeout(() => console.log('C'), 0);
  Promise.resolve().then(() => console.log('D'));
});
console.log('E');
// E B D A C
```

**Scenario:** *"Why does a heavy `for` loop freeze the UI even inside a `.then`?"* Microtasks run on the main thread and block rendering. Split the work into chunks (`setTimeout`, `scheduler.yield()`, `requestIdleCallback`) or move it to a **Web Worker**.

---

### 20.4 Promises & async/await

**Explanation**
A Promise represents a future value in one of three states: **pending → fulfilled | rejected** (settled once, never again).

```js
// Combinators
await Promise.all([a(), b()]);        // all fulfil → array; ANY reject → rejects immediately (fail-fast)
await Promise.allSettled([a(), b()]); // never rejects; [{status:'fulfilled',value}|{status:'rejected',reason}]
await Promise.race([fetchX(), timeout(5000)]);  // first to SETTLE (fulfil or reject)
await Promise.any([mirror1(), mirror2()]);      // first to FULFIL; all reject → AggregateError

// async/await error handling
async function load() {
  try {
    const [user, posts] = await Promise.all([getUser(), getPosts()]);  // parallel ✅
    return { user, posts };
  } catch (e) {
    console.error(e);
    throw e;
  } finally {
    setLoading(false);
  }
}

// ❌ accidental sequential
const user = await getUser();
const posts = await getPosts();   // waits for user unnecessarily

// ❌ forEach doesn't await
items.forEach(async i => await save(i));     // fire-and-forget
// ✅
for (const i of items) await save(i);        // sequential
await Promise.all(items.map(save));          // parallel
```

**Timeout helper:**
```js
const timeout = (ms) => new Promise((_, rej) => setTimeout(() => rej(new Error('Timeout')), ms));
const data = await Promise.race([fetch(url), timeout(5000)]);
// modern: fetch(url, { signal: AbortSignal.timeout(5000) })
```

**Concurrency limit (asked at senior level):**
```js
async function mapLimit(items, limit, fn) {
  const results = [];
  let i = 0;
  async function worker() {
    while (i < items.length) {
      const idx = i++;
      results[idx] = await fn(items[idx]);
    }
  }
  await Promise.all(Array.from({ length: Math.min(limit, items.length) }, worker));
  return results;
}
// await mapLimit(urls, 3, u => fetch(u).then(r => r.json()));
```

---

### 20.5 Prototypes & Inheritance

**Explanation**
Every object has an internal link `[[Prototype]]` (reachable via `Object.getPrototypeOf` / `__proto__`). Property lookups **walk up the prototype chain** until they reach `null`. `class` is **syntax sugar** over prototypes.

```js
function Animal(name) { this.name = name; }
Animal.prototype.speak = function () { return `${this.name} makes a sound`; };

function Dog(name) { Animal.call(this, name); }
Dog.prototype = Object.create(Animal.prototype);
Dog.prototype.constructor = Dog;
Dog.prototype.speak = function () { return `${this.name} barks`; };

// Same with class
class Animal2 { constructor(n) { this.name = n; } speak() { return `${this.name} makes a sound`; } }
class Dog2 extends Animal2 { speak() { return `${super.speak()} (woof)`; } }

const d = new Dog2('Rex');
d.hasOwnProperty('name');   // true (own)
'speak' in d;               // true (inherited)
Object.getPrototypeOf(d) === Dog2.prototype;  // true
```

**What `new` does:** creates an object → links its prototype to `Fn.prototype` → runs `Fn` with `this` = that object → returns it (unless `Fn` returns an object).

---

### 20.6 `var` / `let` / `const`

**Explanation**
| | `var` | `let` | `const` |
|---|---|---|---|
| Scope | function | block | block |
| Hoisting | yes, initialised `undefined` | yes, TDZ | yes, TDZ |
| Redeclare | ✅ | ❌ | ❌ |
| Reassign | ✅ | ✅ | ❌ |
| Global → `window` property | ✅ | ❌ | ❌ |

`const` stops **reassignment**, not **mutation**: `const arr = []; arr.push(1)` is fine. Use `Object.freeze` for a shallow freeze.

**Default style:** `const` everywhere, `let` when you need to reassign, never `var`.

---

### 20.7 Shallow vs Deep Copy & Immutability

**Explanation**
- **Shallow copy** copies the top level; nested objects are **still shared**: `{...obj}`, `[...arr]`, `Object.assign`, `arr.slice()`.
- **Deep copy** copies everything: `structuredClone(obj)` (modern; handles Date, Map, Set and cycles, but not functions or DOM nodes). `JSON.parse(JSON.stringify(obj))` loses Dates, `undefined`, functions, Map/Set and breaks on cycles.

```js
const state = { user: { name: 'A', tags: ['x'] } };
const shallow = { ...state };
shallow.user.name = 'B';
console.log(state.user.name);   // "B": shared!

const deep = structuredClone(state);
deep.user.name = 'C';
console.log(state.user.name);   // "B": unaffected

// Immutable nested update (React style)
const next = { ...state, user: { ...state.user, tags: [...state.user.tags, 'y'] } };

// Non-mutating array methods (ES2023)
arr.toSorted(); arr.toReversed(); arr.toSpliced(1, 1); arr.with(0, 'new');
```

**Why React cares:** React detects changes with **reference equality**. If you mutate, the reference stays the same, so React sees no change and **doesn't re-render** (and memoised selectors break too). **Immer** lets you write mutating code that produces immutable updates.

**Deep clone implementation (asked often):**
```js
function deepClone(value, seen = new WeakMap()) {
  if (value === null || typeof value !== 'object') return value;
  if (value instanceof Date) return new Date(value);
  if (value instanceof RegExp) return new RegExp(value);
  if (seen.has(value)) return seen.get(value);              // handle cycles
  const out = Array.isArray(value) ? [] : Object.create(Object.getPrototypeOf(value));
  seen.set(value, out);
  for (const key of Reflect.ownKeys(value)) out[key] = deepClone(value[key], seen);
  return out;
}
```

---

### 20.8 Debounce & Throttle Implementations

**Explanation**
```js
// Debounce with leading option and cancel
function debounce(fn, wait, { leading = false } = {}) {
  let timer = null;
  function debounced(...args) {
    const callNow = leading && !timer;
    clearTimeout(timer);
    timer = setTimeout(() => {
      timer = null;
      if (!leading) fn.apply(this, args);
    }, wait);
    if (callNow) fn.apply(this, args);
  }
  debounced.cancel = () => { clearTimeout(timer); timer = null; };
  return debounced;
}

// Throttle (leading + trailing)
function throttle(fn, wait) {
  let last = 0, timer = null, lastArgs, lastThis;
  return function (...args) {
    const now = Date.now();
    const remaining = wait - (now - last);
    lastArgs = args; lastThis = this;
    if (remaining <= 0) {
      clearTimeout(timer); timer = null;
      last = now;
      fn.apply(lastThis, lastArgs);
    } else if (!timer) {
      timer = setTimeout(() => {
        last = Date.now(); timer = null;
        fn.apply(lastThis, lastArgs);         // trailing call with latest args
      }, remaining);
    }
  };
}
```

| Debounce | Throttle |
|---|---|
| Waits for a **pause** | Runs at a **steady rate** |
| Search box, autosave, resize end | Scroll position, mouse move, infinite-scroll checks |

---

### 20.9 Polyfills: `map`, `filter`, `reduce`, `Promise.all`, `flat`

**Explanation**
```js
Array.prototype.myMap = function (cb, thisArg) {
  const out = new Array(this.length);
  for (let i = 0; i < this.length; i++) if (i in this) out[i] = cb.call(thisArg, this[i], i, this);
  return out;
};

Array.prototype.myFilter = function (cb, thisArg) {
  const out = [];
  for (let i = 0; i < this.length; i++) if (i in this && cb.call(thisArg, this[i], i, this)) out.push(this[i]);
  return out;
};

Array.prototype.myReduce = function (cb, ...init) {
  let i = 0, acc;
  if (init.length) acc = init[0];
  else {
    while (i < this.length && !(i in this)) i++;
    if (i >= this.length) throw new TypeError('Reduce of empty array with no initial value');
    acc = this[i++];
  }
  for (; i < this.length; i++) if (i in this) acc = cb(acc, this[i], i, this);
  return acc;
};

function flatten(arr, depth = Infinity) {
  return depth > 0
    ? arr.reduce((acc, v) => acc.concat(Array.isArray(v) ? flatten(v, depth - 1) : v), [])
    : arr.slice();
}

Promise.myAll = function (iterable) {
  return new Promise((resolve, reject) => {
    const items = Array.from(iterable);
    const results = new Array(items.length);
    let remaining = items.length;
    if (remaining === 0) return resolve(results);
    items.forEach((p, i) => {
      Promise.resolve(p).then(v => {
        results[i] = v;                  // keep order
        if (--remaining === 0) resolve(results);
      }, reject);
    });
  });
};
```

**Other common ones:** `Function.prototype.bind` ([20.2](#202-this-call-apply-bind)), `memoize`, `curry`, `EventEmitter`, `once`, `compose/pipe`, `deepEqual`.

```js
const curry = fn => function curried(...a) { return a.length >= fn.length ? fn.apply(this, a) : (...b) => curried.apply(this, [...a, ...b]); };
const memoize = fn => { const cache = new Map(); return arg => cache.has(arg) ? cache.get(arg) : (cache.set(arg, fn(arg)), cache.get(arg)); };
const once = fn => { let done = false, r; return (...a) => done ? r : (done = true, r = fn(...a)); };
const pipe = (...fns) => x => fns.reduce((v, f) => f(v), x);

class EventEmitter {
  #events = new Map();
  on(e, fn) { (this.#events.get(e) ?? this.#events.set(e, new Set()).get(e)).add(fn); return () => this.off(e, fn); }
  off(e, fn) { this.#events.get(e)?.delete(fn); }
  emit(e, ...args) { this.#events.get(e)?.forEach(fn => fn(...args)); }
  once(e, fn) { const off = this.on(e, (...a) => { off(); fn(...a); }); }
}
```

---

### 20.10 ES6+ Features & Equality

**Explanation**
```js
// Destructuring with defaults & rename
const { name: userName = 'Guest', address: { city } = {} } = user;
const [first, , third = 0, ...rest] = arr;

// Spread / rest
const merged = { ...defaults, ...options };
function sum(...nums) { return nums.reduce((a, b) => a + b, 0); }

// Optional chaining & nullish coalescing
const zip = user?.address?.zip ?? 'N/A';
user.onSave?.();
// ?? only falls back for null/undefined; || also for 0, '', false
0 || 10;   // 10
0 ?? 10;   // 0

// Logical assignment
opts.timeout ??= 3000;

// Template literals, arrow fns, classes, modules, Map/Set, Symbol, generators, Array.at, Object.groupBy
const grouped = Object.groupBy(people, p => p.dept);
arr.at(-1);  // last element
```

**Equality:**
```js
1 == '1';             // true (type coercion)
1 === '1';            // false
null == undefined;    // true
null === undefined;   // false
NaN === NaN;          // false
Object.is(NaN, NaN);  // true   ← React uses Object.is
Object.is(0, -0);     // false
[] == false;          // true (!), always use ===
{} === {};            // false (different references)
```

**Truthy/falsy:** falsy = `false, 0, -0, 0n, '', null, undefined, NaN`. Everything else is truthy, including `[]`, `{}` and `'0'`.

---

## 21. Machine Coding / Live Coding

### 21.0 How to Approach a Machine Coding Round

**Explanation**
Interviewers grade **process** as much as the result. Follow this order:

1. **Clarify (2–3 min):** inputs, edge cases, must-have vs nice-to-have, API shape, styling expectations, whether you can use libraries.
2. **Plan (2 min):** say the component tree and the state shape out loud. *"I'll have `App` → `SearchInput` + `ResultList`. State: `query`, `results`, `status`."*
3. **Make it work first:** the simplest version that runs.
4. **Then cover:** loading / error / empty states → edge cases → keyboard and accessibility → performance.
5. **Talk as you go**, and name your trade-offs: *"I'd use React Query in production, but I'll write the fetch by hand here to show the race-condition handling."*
6. **Leave time to test** it by hand, and say what you'd add with more time (tests, virtualization, a11y polish).

**Common evaluation points:** working code · sensible component split · state kept minimal and in the right place · keys · no stale closures · cleanup of timers and listeners · a11y basics · readable naming.

---

### 21.1 Debounced Autocomplete with Keyboard Navigation ⭐

**Requirements:** debounce input, fetch suggestions, cancel stale requests, arrow keys and Enter to choose, Escape to close, highlight the match, loading/empty/error states, accessible combobox.

```jsx
function useDebounce(value, delay = 300) {
  const [v, setV] = useState(value);
  useEffect(() => { const t = setTimeout(() => setV(value), delay); return () => clearTimeout(t); }, [value, delay]);
  return v;
}

function highlight(text, query) {
  const i = text.toLowerCase().indexOf(query.toLowerCase());
  if (i === -1 || !query) return text;
  return <>{text.slice(0, i)}<mark>{text.slice(i, i + query.length)}</mark>{text.slice(i + query.length)}</>;
}

export default function Autocomplete({ fetchSuggestions, onSelect }) {
  const [query, setQuery] = useState('');
  const [items, setItems] = useState([]);
  const [status, setStatus] = useState('idle');     // idle | loading | success | error
  const [open, setOpen] = useState(false);
  const [active, setActive] = useState(-1);
  const debounced = useDebounce(query.trim(), 300);
  const cache = useRef(new Map());
  const listId = useId();

  useEffect(() => {
    if (!debounced) { setItems([]); setStatus('idle'); return; }
    if (cache.current.has(debounced)) { setItems(cache.current.get(debounced)); setStatus('success'); return; }

    const ctrl = new AbortController();
    setStatus('loading');
    fetchSuggestions(debounced, ctrl.signal)
      .then(res => { cache.current.set(debounced, res); setItems(res); setStatus('success'); setActive(-1); })
      .catch(e => { if (e.name !== 'AbortError') setStatus('error'); });
    return () => ctrl.abort();
  }, [debounced, fetchSuggestions]);

  const choose = item => { setQuery(item.label); setOpen(false); onSelect?.(item); };

  const onKeyDown = e => {
    if (!open || !items.length) return;
    if (e.key === 'ArrowDown') { e.preventDefault(); setActive(i => (i + 1) % items.length); }
    else if (e.key === 'ArrowUp') { e.preventDefault(); setActive(i => (i - 1 + items.length) % items.length); }
    else if (e.key === 'Enter' && active >= 0) { e.preventDefault(); choose(items[active]); }
    else if (e.key === 'Escape') setOpen(false);
  };

  return (
    <div className="autocomplete">
      <input
        role="combobox"
        aria-expanded={open}
        aria-controls={listId}
        aria-autocomplete="list"
        aria-activedescendant={active >= 0 ? `${listId}-${active}` : undefined}
        value={query}
        onChange={e => { setQuery(e.target.value); setOpen(true); }}
        onKeyDown={onKeyDown}
        onBlur={() => setTimeout(() => setOpen(false), 100)}   // allow click on option
        placeholder="Search…"
      />
      {open && debounced && (
        <ul id={listId} role="listbox">
          {status === 'loading' && <li aria-live="polite">Loading…</li>}
          {status === 'error' && <li role="alert">Failed to load</li>}
          {status === 'success' && items.length === 0 && <li>No results</li>}
          {status === 'success' && items.map((item, i) => (
            <li key={item.id} id={`${listId}-${i}`} role="option" aria-selected={i === active}
                className={i === active ? 'active' : ''}
                onMouseDown={e => e.preventDefault()}   // keep focus on input
                onClick={() => choose(item)}
                onMouseEnter={() => setActive(i)}>
              {highlight(item.label, debounced)}
            </li>
          ))}
        </ul>
      )}
    </div>
  );
}

// usage: the fetcher must be stable (module-level or useCallback)
const fetchCities = (q, signal) => fetch(`/api/cities?q=${encodeURIComponent(q)}`, { signal }).then(r => r.json());
<Autocomplete fetchSuggestions={fetchCities} onSelect={c => console.log(c)} />
```

**Talking points:** debouncing, caching results per query, stale-request cancellation, `aria-activedescendant`, `onMouseDown preventDefault` so the input doesn't blur before the click registers, and the follow-up of virtualizing very long lists.

---

### 21.2 Todo App (useReducer + filters + persistence)

```jsx
const initialTodos = () => JSON.parse(localStorage.getItem('todos') ?? '[]');

function todosReducer(state, action) {
  switch (action.type) {
    case 'add':    return [...state, { id: crypto.randomUUID(), text: action.text, done: false }];
    case 'toggle': return state.map(t => (t.id === action.id ? { ...t, done: !t.done } : t));
    case 'edit':   return state.map(t => (t.id === action.id ? { ...t, text: action.text } : t));
    case 'delete': return state.filter(t => t.id !== action.id);
    case 'clearCompleted': return state.filter(t => !t.done);
    default: return state;
  }
}

const FILTERS = { all: () => true, active: t => !t.done, completed: t => t.done };

export default function TodoApp() {
  const [todos, dispatch] = useReducer(todosReducer, undefined, initialTodos);
  const [filter, setFilter] = useState('all');
  const [text, setText] = useState('');

  useEffect(() => { localStorage.setItem('todos', JSON.stringify(todos)); }, [todos]);

  const visible = todos.filter(FILTERS[filter]);
  const remaining = todos.filter(t => !t.done).length;

  const submit = e => {
    e.preventDefault();
    const value = text.trim();
    if (!value) return;
    dispatch({ type: 'add', text: value });
    setText('');
  };

  return (
    <section>
      <form onSubmit={submit}>
        <label htmlFor="new-todo" className="sr-only">New todo</label>
        <input id="new-todo" value={text} onChange={e => setText(e.target.value)} placeholder="What needs doing?" />
        <button>Add</button>
      </form>

      <ul>
        {visible.map(t => <TodoItem key={t.id} todo={t} dispatch={dispatch} />)}
      </ul>
      {todos.length === 0 && <p>Nothing to do 🎉</p>}

      <footer>
        <span>{remaining} item{remaining !== 1 && 's'} left</span>
        {Object.keys(FILTERS).map(f => (
          <button key={f} aria-pressed={filter === f} onClick={() => setFilter(f)}>{f}</button>
        ))}
        <button onClick={() => dispatch({ type: 'clearCompleted' })}>Clear completed</button>
      </footer>
    </section>
  );
}

const TodoItem = memo(function TodoItem({ todo, dispatch }) {
  const [editing, setEditing] = useState(false);
  const [draft, setDraft] = useState(todo.text);

  const save = () => {
    const v = draft.trim();
    if (v) dispatch({ type: 'edit', id: todo.id, text: v });
    else dispatch({ type: 'delete', id: todo.id });
    setEditing(false);
  };

  return (
    <li>
      <input type="checkbox" checked={todo.done} onChange={() => dispatch({ type: 'toggle', id: todo.id })} aria-label={`Mark ${todo.text} done`} />
      {editing ? (
        <input autoFocus value={draft} onChange={e => setDraft(e.target.value)} onBlur={save}
               onKeyDown={e => { if (e.key === 'Enter') save(); if (e.key === 'Escape') { setDraft(todo.text); setEditing(false); } }} />
      ) : (
        <span onDoubleClick={() => setEditing(true)} style={{ textDecoration: todo.done ? 'line-through' : 'none' }}>{todo.text}</span>
      )}
      <button onClick={() => dispatch({ type: 'delete', id: todo.id })} aria-label={`Delete ${todo.text}`}>✕</button>
    </li>
  );
});
```

**Talking points:** the filtered list is derived, not stored · lazy reducer init · stable `id` keys · `dispatch` is stable, so `memo` on items works · edit mode handles Enter/Escape/blur.

---

### 21.3 Infinite Scroll

```jsx
function useInfiniteList(fetchPage) {
  const [items, setItems] = useState([]);
  const [page, setPage] = useState(1);
  const [hasMore, setHasMore] = useState(true);
  const [loading, setLoading] = useState(false);
  const [error, setError] = useState(null);
  const loadingRef = useRef(false);                 // sync guard against double-fire

  const loadMore = useCallback(async () => {
    if (loadingRef.current || !hasMore) return;
    loadingRef.current = true; setLoading(true); setError(null);
    try {
      const { data, hasMore: more } = await fetchPage(page);
      setItems(prev => [...prev, ...data]);
      setHasMore(more);
      setPage(p => p + 1);
    } catch (e) {
      setError(e);
    } finally {
      loadingRef.current = false; setLoading(false);
    }
  }, [fetchPage, page, hasMore]);

  return { items, loading, error, hasMore, loadMore };
}

export default function Feed() {
  const { items, loading, error, hasMore, loadMore } = useInfiniteList(fetchPosts);
  const sentinelRef = useRef(null);

  useEffect(() => {
    const el = sentinelRef.current;
    if (!el) return;
    const io = new IntersectionObserver(([entry]) => { if (entry.isIntersecting) loadMore(); }, { rootMargin: '300px' });
    io.observe(el);
    return () => io.disconnect();
  }, [loadMore]);

  return (
    <div role="feed" aria-busy={loading}>
      {items.map(p => <article key={p.id}><h3>{p.title}</h3></article>)}
      {error && <button onClick={loadMore}>Failed. Retry</button>}
      {loading && <p>Loading…</p>}
      {!hasMore && <p>You’re all caught up</p>}
      {hasMore && !error && <div ref={sentinelRef} style={{ height: 1 }} />}
    </div>
  );
}

const fetchPosts = page => fetch(`/api/posts?page=${page}&limit=20`).then(r => r.json());
```

**Talking points:** `IntersectionObserver` instead of scroll listeners (better performance) · `rootMargin` to load before reaching the bottom · a ref guard prevents duplicate requests · error retry · combine with virtualization for very long feeds · cursor pagination is better than page numbers when items are inserted.

---

### 21.4 Modal (Accessible, Portal, Focus Trap)

```jsx
const FOCUSABLE = 'a[href],button:not([disabled]),textarea,input,select,[tabindex]:not([tabindex="-1"])';

export function Modal({ open, onClose, title, children }) {
  const dialogRef = useRef(null);
  const titleId = useId();

  useEffect(() => {
    if (!open) return;
    const previouslyFocused = document.activeElement;
    const dialog = dialogRef.current;
    const focusables = () => [...dialog.querySelectorAll(FOCUSABLE)];
    (focusables()[0] ?? dialog).focus();
    document.body.style.overflow = 'hidden';

    const onKeyDown = e => {
      if (e.key === 'Escape') { e.stopPropagation(); onClose(); }
      if (e.key === 'Tab') {
        const els = focusables();
        if (!els.length) return e.preventDefault();
        const first = els[0], last = els[els.length - 1];
        if (e.shiftKey && document.activeElement === first) { e.preventDefault(); last.focus(); }
        else if (!e.shiftKey && document.activeElement === last) { e.preventDefault(); first.focus(); }
      }
    };
    document.addEventListener('keydown', onKeyDown);
    return () => {
      document.removeEventListener('keydown', onKeyDown);
      document.body.style.overflow = '';
      previouslyFocused?.focus();                  // restore focus
    };
  }, [open, onClose]);

  if (!open) return null;
  return createPortal(
    <div className="backdrop" onMouseDown={e => { if (e.target === e.currentTarget) onClose(); }}>
      <div ref={dialogRef} role="dialog" aria-modal="true" aria-labelledby={titleId} tabIndex={-1} className="modal">
        <h2 id={titleId}>{title}</h2>
        {children}
        <button onClick={onClose} aria-label="Close">✕</button>
      </div>
    </div>,
    document.body
  );
}
```

---

### 21.5 Accordion (single / multiple open)

```jsx
export function Accordion({ items, allowMultiple = false }) {
  const [openIds, setOpenIds] = useState(new Set());

  const toggle = id => setOpenIds(prev => {
    const next = new Set(allowMultiple ? prev : []);
    prev.has(id) ? next.delete(id) : next.add(id);
    return next;
  });

  return (
    <div>
      {items.map(({ id, title, content }) => {
        const open = openIds.has(id);
        return (
          <div key={id}>
            <h3>
              <button id={`acc-btn-${id}`} aria-expanded={open} aria-controls={`acc-panel-${id}`} onClick={() => toggle(id)}>
                {title} <span aria-hidden>{open ? '−' : '+'}</span>
              </button>
            </h3>
            <div id={`acc-panel-${id}`} role="region" aria-labelledby={`acc-btn-${id}`} hidden={!open}>{content}</div>
          </div>
        );
      })}
    </div>
  );
}
```
**Note:** when single-open mode toggles the currently open item, `next` is a new empty Set, so `delete` does nothing and the item ends up closed. That's the correct behaviour.

---

### 21.6 Star Rating

```jsx
export function StarRating({ max = 5, value, onChange, readOnly = false }) {
  const [hover, setHover] = useState(0);
  const display = hover || value;

  return (
    <div role="radiogroup" aria-label="Rating" onMouseLeave={() => setHover(0)}>
      {Array.from({ length: max }, (_, i) => {
        const star = i + 1;
        return (
          <button
            key={star}
            type="button"
            role="radio"
            aria-checked={value === star}
            aria-label={`${star} star${star > 1 ? 's' : ''}`}
            disabled={readOnly}
            onMouseEnter={() => !readOnly && setHover(star)}
            onClick={() => onChange(star === value ? 0 : star)}    // click again to clear
            style={{ color: star <= display ? 'gold' : '#ccc', fontSize: 24, background: 'none', border: 0 }}
          >★</button>
        );
      })}
    </div>
  );
}
// <StarRating value={rating} onChange={setRating} />
```
**Extensions:** half stars (check the pointer position inside the star), arrow-key support (roving tabindex).

---

### 21.7 Progress Bar & Countdown Timer

```jsx
export function ProgressBar({ value }) {
  const pct = Math.min(100, Math.max(0, value));
  return (
    <div role="progressbar" aria-valuenow={pct} aria-valuemin={0} aria-valuemax={100} className="track">
      <div className="fill" style={{ transform: `scaleX(${pct / 100})`, transformOrigin: 'left', transition: 'transform 200ms' }} />
      <span>{pct.toFixed(0)}%</span>
    </div>
  );
}

export function Countdown({ seconds: initial, onComplete }) {
  const [endAt, setEndAt] = useState(() => Date.now() + initial * 1000);
  const [remaining, setRemaining] = useState(initial * 1000);
  const [running, setRunning] = useState(true);
  const onCompleteRef = useRef(onComplete);
  useEffect(() => { onCompleteRef.current = onComplete; });

  useEffect(() => {
    if (!running) return;
    const id = setInterval(() => {
      const left = Math.max(0, endAt - Date.now());     // based on timestamps: no drift
      setRemaining(left);
      if (left === 0) { clearInterval(id); setRunning(false); onCompleteRef.current?.(); }
    }, 250);
    return () => clearInterval(id);
  }, [running, endAt]);

  const pause = () => setRunning(false);
  const resume = () => { setEndAt(Date.now() + remaining); setRunning(true); };
  const reset = () => { setRemaining(initial * 1000); setEndAt(Date.now() + initial * 1000); setRunning(true); };

  const total = Math.ceil(remaining / 1000);
  const mm = String(Math.floor(total / 60)).padStart(2, '0');
  const ss = String(total % 60).padStart(2, '0');

  return (
    <div>
      <time aria-live="off">{mm}:{ss}</time>
      {running ? <button onClick={pause}>Pause</button> : <button onClick={resume} disabled={remaining === 0}>Resume</button>}
      <button onClick={reset}>Reset</button>
    </div>
  );
}
```
**Talking point:** compute from `endAt - Date.now()` instead of decrementing by 1 each tick. `setInterval` drifts, and gets throttled heavily in background tabs.

---

### 21.8 Nested Comments / File Explorer (Recursion)

```jsx
const tree = {
  id: 'root', name: 'root', isFolder: true, children: [
    { id: '1', name: 'src', isFolder: true, children: [{ id: '2', name: 'App.jsx', isFolder: false }] },
    { id: '3', name: 'package.json', isFolder: false },
  ],
};

// Pure immutable tree helpers
function insertNode(node, parentId, newNode) {
  if (node.id === parentId) return { ...node, children: [...(node.children ?? []), newNode] };
  if (!node.children) return node;
  return { ...node, children: node.children.map(c => insertNode(c, parentId, newNode)) };
}
function deleteNode(node, id) {
  if (!node.children) return node;
  return { ...node, children: node.children.filter(c => c.id !== id).map(c => deleteNode(c, id)) };
}

export default function FileExplorer() {
  const [data, setData] = useState(tree);
  const add = (parentId, isFolder) => {
    const name = prompt(`New ${isFolder ? 'folder' : 'file'} name`);
    if (name) setData(d => insertNode(d, parentId, { id: crypto.randomUUID(), name, isFolder, children: isFolder ? [] : undefined }));
  };
  const remove = id => setData(d => deleteNode(d, id));
  return <ul role="tree"><Node node={data} onAdd={add} onDelete={remove} depth={0} /></ul>;
}

function Node({ node, onAdd, onDelete, depth }) {
  const [open, setOpen] = useState(depth === 0);
  if (!node.isFolder) {
    return (
      <li role="treeitem" style={{ paddingLeft: depth * 16 }}>
        📄 {node.name} <button onClick={() => onDelete(node.id)} aria-label={`Delete ${node.name}`}>🗑</button>
      </li>
    );
  }
  return (
    <li role="treeitem" aria-expanded={open}>
      <div style={{ paddingLeft: depth * 16 }}>
        <button onClick={() => setOpen(o => !o)}>{open ? '📂' : '📁'} {node.name}</button>
        <button onClick={() => onAdd(node.id, true)}>+ Folder</button>
        <button onClick={() => onAdd(node.id, false)}>+ File</button>
        {depth > 0 && <button onClick={() => onDelete(node.id)}>🗑</button>}
      </div>
      {open && (
        <ul role="group">
          {node.children.map(child => <Node key={child.id} node={child} onAdd={onAdd} onDelete={onDelete} depth={depth + 1} />)}
        </ul>
      )}
    </li>
  );
}
```
**Talking points:** recursive component · immutable tree updates · alternative: **normalised flat map** `{ [id]: { ...node, childIds } }` for O(1) updates in large trees · nested comments use the same structure (reply = insert child).

---

### 21.9 Data Table: Sort, Filter, Paginate

```jsx
export default function DataTable({ rows, columns, pageSize = 10 }) {
  const [query, setQuery] = useState('');
  const [sort, setSort] = useState({ key: null, dir: 'asc' });
  const [page, setPage] = useState(1);

  const filtered = useMemo(() => {
    const q = query.toLowerCase();
    return q ? rows.filter(r => columns.some(c => String(r[c.key]).toLowerCase().includes(q))) : rows;
  }, [rows, columns, query]);

  const sorted = useMemo(() => {
    if (!sort.key) return filtered;
    const factor = sort.dir === 'asc' ? 1 : -1;
    return [...filtered].sort((a, b) => {
      const x = a[sort.key], y = b[sort.key];
      return (typeof x === 'number' ? x - y : String(x).localeCompare(String(y))) * factor;
    });
  }, [filtered, sort]);

  const totalPages = Math.max(1, Math.ceil(sorted.length / pageSize));
  const current = Math.min(page, totalPages);
  const pageRows = sorted.slice((current - 1) * pageSize, current * pageSize);

  const toggleSort = key => {
    setSort(s => (s.key === key ? { key, dir: s.dir === 'asc' ? 'desc' : 'asc' } : { key, dir: 'asc' }));
  };

  return (
    <div>
      <input placeholder="Filter…" value={query} onChange={e => { setQuery(e.target.value); setPage(1); }} aria-label="Filter rows" />
      <table>
        <thead>
          <tr>
            {columns.map(c => (
              <th key={c.key} aria-sort={sort.key === c.key ? (sort.dir === 'asc' ? 'ascending' : 'descending') : 'none'}>
                <button onClick={() => toggleSort(c.key)}>
                  {c.label} {sort.key === c.key ? (sort.dir === 'asc' ? '▲' : '▼') : ''}
                </button>
              </th>
            ))}
          </tr>
        </thead>
        <tbody>
          {pageRows.map(r => <tr key={r.id}>{columns.map(c => <td key={c.key}>{c.render ? c.render(r) : r[c.key]}</td>)}</tr>)}
          {pageRows.length === 0 && <tr><td colSpan={columns.length}>No matching rows</td></tr>}
        </tbody>
      </table>
      <Pagination page={current} totalPages={totalPages} onChange={setPage} />
    </div>
  );
}

function Pagination({ page, totalPages, onChange }) {
  // window of pages with ellipsis: 1 … 4 5 6 … 20
  const pages = [];
  for (let p = 1; p <= totalPages; p++) {
    if (p === 1 || p === totalPages || Math.abs(p - page) <= 1) pages.push(p);
    else if (pages[pages.length - 1] !== '…') pages.push('…');
  }
  return (
    <nav aria-label="Pagination">
      <button disabled={page === 1} onClick={() => onChange(page - 1)}>Prev</button>
      {pages.map((p, i) => p === '…'
        ? <span key={`e${i}`}>…</span>
        : <button key={p} aria-current={p === page ? 'page' : undefined} onClick={() => onChange(p)}>{p}</button>)}
      <button disabled={page === totalPages} onClick={() => onChange(page + 1)}>Next</button>
    </nav>
  );
}
```
**Talking points:** derived data in a `useMemo` chain (filter → sort → slice) · reset to page 1 on filter · `aria-sort` · for large datasets, do sorting/filtering/pagination **on the server** and keep the params in the URL · TanStack Table for production.

---

### 21.10 Tic-Tac-Toe

```jsx
const LINES = [[0,1,2],[3,4,5],[6,7,8],[0,3,6],[1,4,7],[2,5,8],[0,4,8],[2,4,6]];
function getWinner(b) {
  for (const [a, c, d] of LINES) if (b[a] && b[a] === b[c] && b[a] === b[d]) return { player: b[a], line: [a, c, d] };
  return null;
}

export default function TicTacToe() {
  const [history, setHistory] = useState([Array(9).fill(null)]);
  const [step, setStep] = useState(0);
  const board = history[step];
  const xIsNext = step % 2 === 0;                 // derived, not stored
  const win = getWinner(board);
  const isDraw = !win && board.every(Boolean);

  const play = i => {
    if (board[i] || win) return;
    const next = board.slice();
    next[i] = xIsNext ? 'X' : 'O';
    setHistory(h => [...h.slice(0, step + 1), next]);   // drop "future" when time-travelling
    setStep(s => s + 1);
  };

  return (
    <div>
      <p role="status">{win ? `Winner: ${win.player}` : isDraw ? 'Draw!' : `Next: ${xIsNext ? 'X' : 'O'}`}</p>
      <div style={{ display: 'grid', gridTemplateColumns: 'repeat(3, 60px)', gap: 4 }}>
        {board.map((cell, i) => (
          <button key={i} onClick={() => play(i)} aria-label={`Cell ${i + 1}${cell ? `, ${cell}` : ''}`}
                  style={{ height: 60, fontSize: 28, background: win?.line.includes(i) ? '#cfc' : undefined }}>
            {cell}
          </button>
        ))}
      </div>
      <button onClick={() => { setHistory([Array(9).fill(null)]); setStep(0); }}>Restart</button>
      <ol>{history.map((_, m) => <li key={m}><button onClick={() => setStep(m)}>{m ? `Go to move ${m}` : 'Go to start'}</button></li>)}</ol>
    </div>
  );
}
```
**Follow-up:** "Make it N×N with K-in-a-row." Generate the win check by scanning rows, columns and both diagonals from the last move: O(K) per move instead of hard-coded lines.

---

### 21.11 Traffic Light (state machine with timers)

```jsx
const CONFIG = {
  red:    { duration: 4000, next: 'green' },
  green:  { duration: 3000, next: 'yellow' },
  yellow: { duration: 1000, next: 'red' },
};

export default function TrafficLight() {
  const [light, setLight] = useState('red');

  useEffect(() => {
    const t = setTimeout(() => setLight(CONFIG[light].next), CONFIG[light].duration);
    return () => clearTimeout(t);
  }, [light]);

  return (
    <div style={{ display: 'flex', flexDirection: 'column', gap: 8, background: '#222', padding: 12, width: 'fit-content' }}>
      {Object.keys(CONFIG).map(color => (
        <div key={color} aria-label={color} aria-current={light === color}
             style={{ width: 50, height: 50, borderRadius: '50%', background: light === color ? color : '#555' }} />
      ))}
    </div>
  );
}
```
**Talking point:** a config-driven state machine. Adding a light or changing timings means changing only the config, not the logic. `setTimeout` chained on state is easier to reason about than `setInterval` here.

---

### 21.12 Carousel / Image Slider

```jsx
export function Carousel({ slides, autoPlayMs = 4000 }) {
  const [index, setIndex] = useState(0);
  const [paused, setPaused] = useState(false);
  const count = slides.length;
  const go = useCallback(i => setIndex(((i % count) + count) % count), [count]);   // wraps around

  useEffect(() => {
    if (paused || count < 2) return;
    const t = setTimeout(() => go(index + 1), autoPlayMs);
    return () => clearTimeout(t);
  }, [index, paused, autoPlayMs, go, count]);

  return (
    <section aria-roledescription="carousel" aria-label="Featured"
             onMouseEnter={() => setPaused(true)} onMouseLeave={() => setPaused(false)}
             onFocus={() => setPaused(true)} onBlur={() => setPaused(false)}
             onKeyDown={e => { if (e.key === 'ArrowRight') go(index + 1); if (e.key === 'ArrowLeft') go(index - 1); }}>
      <div style={{ overflow: 'hidden' }}>
        <div style={{ display: 'flex', transform: `translateX(-${index * 100}%)`, transition: 'transform 400ms' }}>
          {slides.map((s, i) => (
            <img key={s.id} src={s.src} alt={s.alt} aria-hidden={i !== index}
                 loading={i === 0 ? 'eager' : 'lazy'} style={{ minWidth: '100%' }} />
          ))}
        </div>
      </div>
      <button onClick={() => go(index - 1)} aria-label="Previous slide">‹</button>
      <button onClick={() => go(index + 1)} aria-label="Next slide">›</button>
      <div>
        {slides.map((s, i) => (
          <button key={s.id} aria-label={`Go to slide ${i + 1}`} aria-current={i === index} onClick={() => go(i)}>•</button>
        ))}
      </div>
      <button onClick={() => setPaused(p => !p)}>{paused ? 'Play' : 'Pause'}</button>
    </section>
  );
}
```
**Talking points:** modulo wrap-around · pause on hover/focus (a WCAG requirement for auto-moving content) · lazy-loading images · `prefers-reduced-motion` · swipe support with pointer events.

---

### 21.13 More Problems to Practise (with hints)

| Problem | Key hints |
|---|---|
| **Stopwatch** | timestamp-based like the countdown; `useRef` for the interval; lap list |
| **OTP input** | array of refs; auto-advance on input; Backspace moves back; paste fills all; `inputMode="numeric"` |
| **Multi-step form** | one form state; per-step validation; progress indicator; back keeps the data |
| **Kanban drag & drop** | HTML5 DnD or dnd-kit; state `{ columns: { id: taskIds[] }, tasks: {byId} }`; optimistic move |
| **Toast system** | Context + portal; `toast.success()` API; auto-dismiss timers; stacking; `aria-live` |
| **Dropdown / Select** | click outside; keyboard nav; portal for overflow; `aria-expanded` |
| **Tabs** | compound components ([5.3](#53-compound-components)); arrow-key roving tabindex |
| **Chips / tag input** | Enter/comma to add, Backspace on empty removes the last one, de-duplicate |
| **Typeahead with cache + LRU** | `Map` with a size limit; delete + re-set to update recency |
| **Polling dashboard** | `setTimeout` loop after each response (not `setInterval`); pause when the tab is hidden (`visibilitychange`) |
| **Image gallery with lightbox** | grid; modal; arrow keys; preload neighbouring images |
| **Virtualized list from scratch** | `scrollTop / rowHeight` → start index; render a window; spacer height |
| **Undo/redo** | `{ past: [], present, future: [] }` reducer |
| **Snake / Memory game** | game loop with `setInterval` + ref for direction; derived collision checks |

---

## 22. Behavioural & Experience

### 22.1 The STAR Method

**Explanation**
Answer experience questions with **STAR**:
- **S**ituation: context (1–2 sentences).
- **T**ask: your responsibility / the goal.
- **A**ction: **what *you* did** (say "I", not "we"), including the technical detail and the trade-offs. This should be about 60% of the answer.
- **R**esult: **measurable** outcome (numbers!) + what you learned.

Keep each story **2–3 minutes**. Prepare **5–6 stories** that you can adapt to many questions.

---

### 22.2 "Walk Me Through Your Project's Frontend Architecture"

**Template answer structure (fill in your own details):**
1. **Product & scale**: what it does, users, team size, your role.
2. **Stack**: React version, framework (Next/Vite), TypeScript, styling, state, data fetching.
3. **Rendering**: CSR/SSR/SSG, and why.
4. **State & data flow**: server state (React Query), client state (Zustand/Redux), URL state, forms.
5. **Structure**: feature-based folders, design system, monorepo?
6. **Quality**: testing strategy, CI/CD, code review, lint/TS strictness.
7. **Performance & observability**: Web Vitals, monitoring, error tracking.
8. **One thing you'd change** and why (shows you can judge your own work).

**Example:**
> "It's a B2B analytics dashboard: ~20k daily users, 6 frontend engineers; I'm the frontend lead. We use React 18 with Vite and TypeScript strict mode. It's behind login, so we chose CSR plus code splitting by route. Server state is in React Query with a 30s `staleTime`; the only global client state is auth and the user's preferences, in Zustand; filters live in the URL so reports can be shared. Folders are feature-sliced with a shared design system built on Radix and Tailwind, documented in Storybook. Testing: Vitest + RTL + MSW for integration tests, Playwright for 15 critical flows, Chromatic for visual regression. Sentry and Datadog RUM watch errors and INP. If I started again, I'd use SSR for the public share pages: unfurling links in Slack doesn't work today because the page is CSR."

---

### 22.3 "Tell Me About a Performance Problem You Solved"

**Example (adapt to your experience):**
> **S:** Our order management table (2,000 rows, inline editing) took ~800ms to respond to each keystroke; INP was in the "poor" range.
> **T:** I owned getting it under 200ms without removing features.
> **A:** I profiled with React DevTools and found every row re-rendering on each keystroke: the edit state lived at the table level and each row got inline callbacks. I moved the edit state into the row, memoised the rows with stable callbacks (`useCallback` + ID-based handlers), and virtualized the table with TanStack Virtual so only ~30 rows were rendered. I also moved the totals calculation into `useMemo` keyed on the data version.
> **R:** Interaction time dropped from 800ms to ~40ms, INP p75 went from 650ms to 120ms, and support tickets about "laggy table" stopped. I wrote a short internal guide on profiling that two other teams used.

---

### 22.4 "Tell Me About a Hard Bug You Debugged"

**Example:**
> **S:** Users occasionally saw another customer's details for a moment when switching quickly between customers in the CRM.
> **T:** It was a P1 (privacy risk); I took ownership.
> **A:** I reproduced it by throttling the network: request A (slow) resolved after request B and overwrote the state, a **race condition** in a `useEffect` fetch. I fixed it straight away with an `AbortController` cleanup, wrote a regression test using MSW with delayed responses, then led the migration of that module to React Query (keyed caching removes this whole class of bug). I also added an ESLint rule warning about fetch-in-effect without cleanup.
> **R:** Zero reports of it since; the module's data code shrank by ~40%; the pattern became our team standard.

---

### 22.5 Technical Decisions & Trade-offs

**Common questions:** *"Describe a technical decision you made and its trade-offs." / "Tell me about a time you chose not to use a popular technology."*

**What a strong answer includes:**
- The **options** you considered (at least two).
- **Criteria**: team skills, performance, maintainability, cost, time to market.
- **Why** you chose this option, **what you gave up**, and **how you reduced the risk** (spike, proof of concept, feature flag, rollback plan).
- What you'd **revisit** later.

**Example:**
> "For our new admin portal, the team wanted Redux because it was familiar. I suggested React Query plus Zustand: 80% of our state was server data, and Redux would have meant writing caching, loading flags and invalidation by hand. The trade-off was that people had to learn something new and we lost Redux DevTools time-travel. To reduce the risk, I built one feature both ways as a spike: the React Query version was 60% less code. We documented the patterns, and after 6 months onboarding was faster and we had fewer stale-data bugs."

---

### 22.6 Code Review, Mentoring & Disagreements

**Code review: what do you look for?**
Correctness and edge cases (loading, error, empty) → accessibility → performance problems (unneeded effects, missing keys, re-render causes) → security (`dangerouslySetInnerHTML`, secrets) → test coverage of behaviour → readability and naming → consistency with team patterns. Be kind and specific, separate "blocking" from "nit", explain the *why*, and suggest code.

**Mentoring example:**
> "A junior dev's PRs kept getting large review comment threads about effects. Instead of fixing them in review, I paired with them for an hour on 'You might not need an effect', and we refactored one of their components together. I then gave them ownership of a small feature with a check-in every two days. In two months their PRs needed half the revisions, and they presented the pattern at our frontend guild meeting."

**Disagreement example:**
> "A senior backend engineer wanted the frontend to poll every 2s for job status. I was concerned about server load and battery. I didn't argue opinions; I collected data: 5k concurrent users × 0.5 req/s = 2.5k req/s just for polling. I proposed SSE, with polling as a fallback. We agreed to an experiment behind a flag, and SSE cut the requests by 95%. I made sure we presented it as a joint decision."

---

### 22.7 Questions to Ask the Interviewer

- What does the frontend architecture look like? React version, SSR or not?
- How is frontend quality measured: Web Vitals, error budgets, test coverage?
- How do frontend and backend/design teams work together? Is there a design system?
- What's the biggest frontend technical challenge right now?
- How are decisions about new libraries or patterns made?
- What does success look like for this role in the first 6 months?
- How is on-call / production support handled for the frontend?

---

> **Final tip:** For every topic above, practise saying the **one-paragraph explanation out loud**, then the **scenario answer**, then a **real example from your own work**. Interviewers remember candidates who connect concepts to real production experience.
