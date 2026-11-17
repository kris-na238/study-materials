# React Interview — Complete Topic Guide

> Every major React interview topic and subtopic with an interview-friendly explanation, code, and a scenario-based example.
>

**How each subtopic is laid out**
- **Explanation**: what to say in the interview
- **Code**: the smallest example that proves you understand it
- **Scenario**: a "what would you do if…" question and the answer
- **Tip / Follow-up**: what interviewers dig into next
- **Interview Questions**: every section ends with the most frequently asked questions and answers (⭐ = asked very often)

---

## Table of Contents

**[The Big Picture](#the-big-picture)**: how all the topics connect

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
23. [HTML & CSS Fundamentals](#23-html--css-fundamentals)
24. [Browser Internals](#24-browser-internals)
25. [Network & Data Fetching Fundamentals](#25-network--data-fetching-fundamentals)
26. [Memory Leaks](#26-memory-leaks)
27. [Web Workers, Service Workers & PWA](#27-web-workers-service-workers--pwa)
28. [Practical Features](#28-practical-features)
29. [Frontend System Design Case Studies](#29-frontend-system-design-case-studies)

**[Revision Cheat Sheet](#revision-cheat-sheet)**: one line per topic

---

## The Big Picture

Think of frontend interviews as **one journey: code becomes a page, the user interacts, and the UI updates**. Every topic in this guide is a stop on that journey, or a layer that supports it.

```
 ┌─ INTERVIEW SKILLS ────────────────────────────────────────────────────────────────┐
 │  Machine coding (21) · Frontend system design (19, 29) · Behavioural (22)         │
 └───────────────────────────────────────────────────────────────────────────────────┘
 ┌─ QUALITY (applies to everything) ─────────────────────────────────────────────────┐
 │  Testing (14) · Security (17) · Accessibility (18) · TypeScript (16)              │
 └───────────────────────────────────────────────────────────────────────────────────┘
 ┌─ THE JOURNEY ─────────────────────────────────────────────────────────────────────┐
 │  BUILD ───────► DELIVER ───────► RENDER ───────► INTERACT ───────► UPDATE         │
 │  Tooling (19)   SSR/RSC (10)     Components (1)  Events (1, 24)    State (6)      │
 │  Bundling (7)   Caching (25)     Hooks (3)       Forms (12)        Data (6, 25)   │
 │                 Hydration (10)   Rendering (2)   Routing (11)      Concurrency (8)│
 │                                  Styling (15)    Features (28)     Errors (13)    │
 │                     ◄─────────── Performance (7) · Memory (26) ──────────►        │
 └───────────────────────────────────────────────────────────────────────────────────┘
 ┌─ FOUNDATIONS ─────────────────────────────────────────────────────────────────────┐
 │  JavaScript (20) · HTML & CSS (23) · Browser (24) · Network (25) · Workers (27)   │
 └───────────────────────────────────────────────────────────────────────────────────┘
```

**The one idea behind React:** *UI is a function of state. You change state, React works out what changed, and the browser paints it.* Nearly every question is about one part of that loop:

```
event ──► setState ──► render (pure) ──► reconcile ──► commit to DOM ──► browser paints ──► Effects run
  ▲                                                                                          │
  └──────────────────────────────── user sees the result ◄───────────────────────────────────┘
```

**Threads that connect the topics**

1. **The update loop.** `useState` ([3.1](#31-usestate)) → what triggers a render ([2.3](#23-what-triggers-a-re-render)) → render and commit ([2.4](#24-render-phase-vs-commit-phase)) → reconciliation ([2.2](#22-reconciliation-heuristics-position-type-key)) → browser pipeline ([24.1](#241-the-critical-rendering-path)).
2. **The identity thread.** Keys ([1.5](#15-rendering-lists--keys)) → reconciliation ([2.2](#22-reconciliation-heuristics-position-type-key)) → a change of type or key resets state → that's why index keys cause bugs.
3. **The closure thread.** Closures ([20.1](#201-closures-scope--hoisting)) → stale closures ([3.11](#311-stale-closures)) → dependency arrays ([3.2](#32-useeffect)) → `useEffectEvent` ([9.6](#96-react-192-activity--useeffectevent)) → timer bugs in machine coding ([21.7](#217-progress-bar--countdown-timer)).
4. **The re-render thread.** Why it renders ([2.3](#23-what-triggers-a-re-render)) → move state down ([7.3](#73-avoiding-unnecessary-re-renders-structural-techniques)) → `React.memo` ([7.2](#72-reactmemo)) → `useMemo`/`useCallback` ([3.8](#38-usememo--usecallback)) → React Compiler ([7.11](#711-react-compiler)).
5. **The main-thread thread.** Event loop ([20.3](#203-the-event-loop-microtasks-vs-macrotasks)) → long tasks → transitions ([8.2](#82-usetransition--starttransition)) → Web Workers ([27.1](#271-web-workers)) → INP ([7.9](#79-web-vitals-lcp-inp-cls)).
6. **The server-to-client thread.** CSR vs SSR ([10.1](#101-csr-vs-ssr-vs-ssg-vs-isr)) → hydration ([10.3](#103-hydration--hydration-mismatches)) → Server Components ([10.2](#102-react-server-components-rsc-vs-client-components)) → streaming ([8.5](#85-streaming-ssr)) → Server Actions ([9.5](#95-server-actions)) → their security risks ([17.6](#176-react-server-components-vulnerability-cve-2025-55182)).
7. **The data thread.** Fetching ([25.4](#254-making-data-fetching-robust)) → race conditions ([3.2](#32-useeffect)) → server state ([6.5](#65-react-query--swr-server-state)) → HTTP caching ([25.2](#252-http-caching)) → Suspense ([8.4](#84-suspense)) → optimistic updates ([9.2](#92-useoptimistic)).
8. **The cleanup thread.** Effect cleanup ([3.2](#32-useeffect)) → StrictMode double-run ([2.7](#27-strictmode)) → memory leaks ([26.1](#261-what-causes-memory-leaks-in-react-apps)) → `<Activity>` ([9.6](#96-react-192-activity--useeffectevent)).

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

**Quick comparison**

| Aspect | Controlled | Uncontrolled |
|---|---|---|
| Source of truth | React state | DOM |
| Value prop | `value` / `checked` | `defaultValue` / `defaultChecked` |
| Read value with | state variable | `ref.current.value` / `FormData` |
| Re-render per keystroke | Yes | No |

**More gotchas**
- **`value` without `onChange`** makes the input read-only, and React warns. Add `onChange`, or use `readOnly` / `defaultValue`.
- **`defaultValue` only sets the initial value.** Changing it later does nothing. To re-initialize, remount the input with a new `key`.
- **Checkboxes and radios** use `checked` / `defaultChecked` and are read with `.checked`, not `.value`.
- **`ref.current` is `null` during the first render.** Read it only in event handlers or effects. Changing a ref never causes a re-render, so don't render `ref.current.value` in JSX.
- **Libraries:** React Hook Form is mostly uncontrolled (refs, fast). Formik is controlled (state).

**Uncontrolled forms with many inputs** (see [3.5 `useRef`](#35-useref))

Don't call `useRef` once per field, and never inside a loop (Rules of Hooks). Pick one of these:

```jsx
// A) One ref on the <form> + FormData: inputs only need a `name`
const formRef = useRef(null);
const submit = e => {
  e.preventDefault();
  const fd = new FormData(formRef.current);   // or new FormData(e.target)
  const data = Object.fromEntries(fd);
  data.skills = fd.getAll('skills');           // checkbox group / multi-select
  formRef.current.reset();                     // clears every field
};
// Unchecked checkboxes are left out of FormData entirely.

// B) One ref object + callback refs (how React Hook Form's register() works)
const inputs = useRef({});
const register = name => el => {
  if (el) inputs.current[name] = el;          // on mount
  else delete inputs.current[name];           // on unmount (el === null)
};
// <input ref={register('email')} />
// On error: inputs.current.email.focus()

// C) Dynamic list: an array of refs (OTP boxes, repeated fields)
const refs = useRef([]);
// {Array.from({ length: 6 }).map((_, i) =>
//   <input key={i} maxLength={1} ref={el => (refs.current[i] = el)}
//          onChange={e => e.target.value && refs.current[i + 1]?.focus()} />)}
// const otp = refs.current.map(el => el.value).join('');
```

**Mixing both:** keep values the UI depends on in state (controlled), and add a `ref` only for DOM work such as `focus()`, `scrollIntoView()` or reading `files[0]`.

**At component level:** the same idea applies to your own components. `<Modal isOpen={open} onClose={...} />` is controlled because the parent owns the state, while a modal that tracks its own `isOpen` is uncontrolled. Good components support both: they use `value` if it's passed and otherwise fall back to internal state from `defaultValue` (see [5.5](#55-controlled-component-pattern--prop-getters)).

**One-line answer:** *"Controlled means React state drives the input through `value` + `onChange`, so React is the single source of truth. Uncontrolled means the DOM keeps the value and we read it through a `ref` or `FormData` on submit. I use controlled inputs for live validation and dynamic UI, and uncontrolled ones for simple or large forms and file inputs. For many uncontrolled fields I use one form ref with `FormData`, or a single ref object filled by callback refs."*

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

**Lifting state up: before and after**

Data only flows down, so siblings can't share state with each other directly. Move the state to their common parent, then pass the **value** down and a **setter callback** down too, so a child can send changes back up.

```jsx
// ❌ Before: SearchBar owns `query`, so ResultsList can't see it
function SearchBar() { const [query, setQuery] = useState(''); /* ... */ }

// ✅ After: the parent is the single source of truth
function SearchPage() {
  const [query, setQuery] = useState('');
  return (
    <>
      <SearchBar query={query} onQueryChange={setQuery} />  {/* sends changes up */}
      <ResultsList query={query} />                          {/* reads the value */}
    </>
  );
}
function SearchBar({ query, onQueryChange }) {
  return <input value={query} onChange={e => onQueryChange(e.target.value)} />;
}
```
The user types, `onQueryChange` fires, the parent's state updates, and both children re-render with the new `query`. `SearchBar` is now a **controlled component** (see 1.8).

**Rule:** keep state *as low as possible, but as high as necessary*. Lifting it too high makes unrelated parts of the page re-render. The reverse move, **moving state down**, is a performance fix (see [7.3](#73-avoiding-unnecessary-re-renders-structural-techniques)).

**Why prop drilling hurts**

```jsx
function App()           { const [user] = useState({ name: 'Krishna' }); return <Layout user={user} />; }
function Layout({ user }) { return <Sidebar user={user} />; }   // doesn't use it
function Sidebar({ user }) { return <Avatar user={user} />; }   // doesn't use it
function Avatar({ user })  { return <img alt={user.name} />; }  // the only real user
```
- The middle components are **coupled** to data they don't care about.
- Renaming a prop or adding a field means editing **every level**.
- The middle components **re-render** whenever the prop changes.
- It's harder to trace where a prop is actually used.

Drilling through **1–2 levels is fine and normal**. It's only a problem when it goes deep or repeats in many places.

**Why composition comes first:** there's no new API, data flow stays explicit, and there's a performance benefit: an element passed as `children` was created by the parent, so it doesn't re-render when the wrapper's own state changes (see 2.3 and 7.3). Context comes next, because every consumer re-renders when its `value` changes, so it suits global data that **rarely changes** (see [3.7](#37-usecontext) and [6.2](#62-context-api--its-limits)).

**Decision summary**

| Situation | Use |
|---|---|
| Siblings need the same data | **Lift state** to the closest common parent |
| Passing through 1–2 levels | Plain props (fine) |
| Middle components only pass data along | **Composition** (`children` or slot props, see [5.8](#58-composition-vs-inheritance)) |
| Many distant components need global data that rarely changes (theme, locale, auth) | **Context** |
| Large or frequently changing global state, needs selective subscriptions | **Zustand / Redux** (see [6.3](#63-redux--redux-toolkit), [6.4](#64-zustand-jotai-recoil-mobx)) |
| Data fetched from the server | **React Query / SWR** (server state, not UI state) |

**One-line answer:** *"Lifting state up means moving shared state to the closest common parent so siblings stay in sync. That can lead to prop drilling, where props pass through components that don't use them. I fix it with composition first, then Context for global data that rarely changes, and a state library for large or frequently changing state."*

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

**Why React wraps events:** the same API in every browser, plus React controls the event system. It can prioritise events (a click is more urgent than a mousemove in concurrent rendering) and make bubbling follow the **React tree**.

**React vs HTML syntax**

| HTML / DOM | React |
|---|---|
| `onclick="handle()"` (lowercase, string) | `onClick={handle}` (camelCase, function) |
| `return false` stops the default action | ❌ doesn't work. Call `e.preventDefault()` |
| `addEventListener('click', fn)` | JSX prop `onClick={fn}` |

**Passing arguments, and the "Too many re-renders" bug**
```jsx
<button onClick={handleClick}>             // ✅ React calls it on click
<button onClick={handleClick()}>           // ❌ runs during render
<button onClick={() => deleteItem(id)}>    // ✅ wrap it to pass arguments
<button onClick={setCount(count + 1)}>     // ❌ sets state during render → infinite loop
```

**How delegation works**
```
Click on <button> → the native event bubbles up the DOM → React's single listener on #root
  → React finds the component path in its fiber tree → builds a SyntheticEvent
  → calls the onClick handlers along that path (bubble order)
```
Benefits: one listener instead of thousands, and elements added later work automatically.

**Capture vs bubble phase:** handlers run in the bubble phase by default. Add `Capture` to the prop name for the capture phase.
```jsx
<div onClickCapture={() => log(1)} onClick={() => log(4)}>
  <button onClickCapture={() => log(2)} onClick={() => log(3)}>Click</button>
</div>
// Order: 1 → 2 → 3 → 4
```

**`stopPropagation` vs `preventDefault`** (often confused)

| Method | What it does | Example |
|---|---|---|
| `e.stopPropagation()` | Stops the event reaching **parent handlers** | A Delete button inside a clickable row |
| `e.preventDefault()` | Stops the **browser's default action** | A form submit reloading the page, a link navigating |

**`e.target` vs `e.currentTarget`:** `target` is what was actually clicked (could be an inner `<span>`). `currentTarget` is the element the handler is attached to. Use `currentTarget` to read the owner's `data-*` attributes.

**Event pooling (React 16 and earlier):** React reused event objects, so after the handler finished, `e.target` was `null` inside `setTimeout` or `await` unless you called `e.persist()`. React 17 removed pooling, and `e.persist()` now does nothing.

**Other differences from the native DOM**
- **`onChange` fires on every keystroke**, like the native `input` event. The native `change` event only fires on blur.
- **Portals:** events bubble through the **React tree**, not the DOM tree. A click inside a portal still reaches its React parent's `onClick` (see [5.6](#56-portals)).
- **`onScroll` doesn't bubble** in React 17+.
- **`window` / `document` events** (resize, keyboard shortcuts, clicks outside an element) need `addEventListener` in `useEffect`, **with cleanup**:
  ```jsx
  useEffect(() => {
    const onResize = () => setWidth(window.innerWidth);
    window.addEventListener('resize', onResize);
    return () => window.removeEventListener('resize', onResize);
  }, []);
  ```

**More scenarios**
- *"My form reloads the page on submit."* Call `e.preventDefault()` in `onSubmit`.
- *"Too many re-renders" on a button.* You wrote `onClick={fn()}` instead of `onClick={fn}` or `onClick={() => fn()}`.

**One-line answer:** *"React passes handlers a SyntheticEvent, a cross-browser wrapper around the native event with the same API. Instead of a listener per element, it uses event delegation: one listener per event type on the root container (since React 17), and bubbling follows the React tree. Pooling was removed in React 17. Handlers take a function reference, `stopPropagation` stops parent handlers, and `preventDefault` stops the browser's default action."*

---

### 1.11 Pure Components & Keeping Render Pure

**Explanation**
A render must be a **pure function** of props + state + context:
- Same input → same JSX output.
- **No side effects during render**: no API calls, no mutating outside variables, no `setState` on other components, no DOM changes.

React may call render **more than once**: StrictMode double-renders in dev, and concurrent rendering can throw a render away and start again. Side effects in render would then run several times.

**Why purity matters:** React relies on it to skip, repeat, pause or reorder renders safely. `React.memo`, `useMemo`, concurrent features and the React Compiler all assume that rendering the same input again gives the same output.

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

// ✅ Local mutation is fine: the array is created during this render
function Grid({ rows }) {
  const cells = [];
  rows.forEach(r => cells.push(<Cell key={r.id} {...r} />));
  return <div>{cells}</div>;
}
```

**Where side effects go instead**

| Side effect | Put it in |
|---|---|
| Caused by a user action (submit, click, track a button) | **Event handler** (preferred) |
| Caused by the component appearing or data changing (fetch, subscribe, "screen viewed") | **`useEffect`** |
| Other non-deterministic values during render (`Math.random()`, `Date.now()`) | Compute them in a handler or effect, or once in `useState(() => ...)` |

**Don't confuse this with `PureComponent` / `React.memo`:** "keeping render pure" is a **rule** for how you write any component. `PureComponent` (class) and `React.memo` (function) are **optimisations** that skip a re-render when a shallow comparison finds the props unchanged. Skipping is only safe *because* render is pure (see 4.3 and 7.2).

**Scenario:** *"An analytics event fires twice in development."* It was called during render, or in an effect without proper handling. StrictMode exposes this. Move it to an event handler (for user actions) or an effect (for "screen viewed"), and make it idempotent (safe to run twice).

---

### 1.12 Interview Questions

**Q1. What is React, and why would you use it?** ⭐
React is a JavaScript library for building UIs from components. You describe what the UI should look like for a given state, and React updates the DOM to match. Its main benefits are reusable components, predictable one-way data flow and a very large ecosystem. It's a library rather than a framework, so routing and data fetching come from other tools or a framework like Next.js.

**Q2. What does JSX compile to?** ⭐
Function calls that create plain objects (React elements). `<div className="a" />` becomes `jsx('div', { className: 'a' })` with the modern JSX transform, or `React.createElement` with the classic one. JSX isn't HTML: it's syntax for describing elements in JavaScript.

**Q3. What's the difference between props and state?** ⭐
Props are passed in by the parent and are read-only. State is owned by the component and changes over time through its setter. Changing either causes a re-render. If a value can be calculated from props or other state, it shouldn't be state at all.

**Q4. Why does React need keys, and why is the index a bad key?** ⭐
Keys tell React which item is which between renders, so it can keep the right state and DOM node for each one. An index changes when items are inserted, removed or reordered, so state attaches to the wrong item, such as text in the wrong input. Use a stable ID from the data. An index is only safe for static lists that never change order.

**Q5. Controlled vs uncontrolled components?** ⭐
In a controlled input, React state holds the value and `onChange` updates it, so validation and formatting are easy. In an uncontrolled input, the DOM holds the value and you read it with a ref or `FormData`, which means less code and fewer re-renders. Use controlled when the UI reacts to each change, and uncontrolled for simple forms or file inputs.

**Q6. What is prop drilling, and how do you avoid it?**
It's passing props through layers that don't use them. Try composition first: pass components as `children` so the data skips the middle layers. Use Context for values many components need, and a state library for complex shared state.

**Q7. Why must a component's render be pure?**
React can call render more than once, pause it or discard the result, especially in concurrent rendering. Side effects during render can therefore run multiple times or at the wrong moment. The same props and state must produce the same output, and side effects belong in event handlers or Effects.

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

**Common misconception: React does NOT read the real DOM to diff**

Both trees in the comparison are **virtual** (JS objects). The old tree is kept in memory from the previous render. The real DOM is only **written to**, during commit.

```
setState → Render: re-run the component → NEW virtual tree
         → Diff: NEW virtual tree vs OLD virtual tree (from memory)
         → Commit: apply only the changes to the real DOM
         → the NEW tree becomes the "old" tree for the next update
```
- React builds a **new** tree rather than editing the old one in place.
- Only the component whose state changed **and its children** re-render, not the whole app (unless a child is memoised, see 7.2).
- In modern React, this "virtual DOM" is the **Fiber tree**. React keeps a *current* tree (on screen) and a *work-in-progress* tree (the new render), and swaps them after commit (see 2.5).

**The tip, explained: why the VDOM isn't faster than the DOM**

```js
// Hand-written: you already know what changed
countEl.textContent = 1;
// React: re-render → build tree → diff → countEl.textContent = 1  (the same final DOM operation + extra work)
```
The VDOM is **overhead**. It's fast compared with re-rendering the whole page (`innerHTML = ...`), not compared with precise manual updates. What you gain is **declarative code** (`UI = f(state)`): you describe the result and React works out the minimal DOM changes. That means fewer bugs from forgotten UI updates, a predictable UI, and performance that's good enough, with `memo` and virtualization for the hot spots.

| Framework | How it updates the DOM |
|---|---|
| **React** | Re-run the component → new VDOM → diff → patch |
| **Svelte** | A **compiler** generates precise DOM updates at build time |
| **Solid** | **Signals**: each piece of state updates exactly the DOM nodes that use it, with no re-running and no diff |

So the VDOM is an **implementation choice**, not the source of speed.

**One-line answer:** *"When state changes, React re-renders the affected components into a new virtual tree, diffs it against the previous virtual tree kept in memory, and applies only the minimal changes to the real DOM during commit. It never reads the real DOM to diff. The VDOM isn't faster than direct DOM updates. Its value is declarative code with good-enough performance, and Svelte and Solid show you can get that without a VDOM."*

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

**Identity decides re-render vs remount**

React asks: *"Is this the same component as last time?"*
- **Same type + position + key** → the same component. React **keeps** it (state + DOM) and **re-renders** it with new props.
- **Any of them changed** → a different component. React **unmounts** the old one and **mounts** a new one, so the **state resets**.

| | **Re-render** | **Remount** |
|---|---|---|
| What happens | The component function is called again | The old instance is destroyed and a new one created |
| State | ✅ Kept | ❌ Lost (back to initial values) |
| DOM nodes | Reused, only the changes patched | Removed and recreated |
| Effects | Re-run only if their deps changed | Cleanup for the old instance, then all effects run fresh |
| Caused by | State change, parent re-render, Context change | A changed type, key or position |

A common misconception: a change of identity doesn't cause a *re-render*, it causes a *remount*. And a component re-renders whenever its parent does, **even when its identity is unchanged** (unless it's memoised). Identity only decides whether React **keeps** the instance or **replaces** it.

**Reordering a list with stable keys** → React matches items by key and **moves** the existing DOM nodes. Nothing is remounted, and each item keeps its own state. With `key={index}`, React matches by **position**, so after a reorder, the instance at position 0 receives another item's data but keeps the old state (the "wrong row stays checked" bug, see 1.5).

**Remounting on purpose:** change the key to reset a component, e.g. `<ProfileForm key={userId} />` gives a fresh form for each user.

**One-line answer:** *"React identifies each element by its type, position and key. If they match the previous render, it keeps the instance and its state and just re-renders it with new props. If any of them changes, it unmounts the old instance and mounts a new one, resetting the state. With stable keys, reordered items are moved, not recreated."*

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

**You call the APIs, Fiber does the scheduling**

`startTransition`, `useTransition` and `useDeferredValue` are **public APIs you use in your own code**. Fiber is the internal engine that carries them out. React can't guess which updates are less important, so by default it treats every `setState` from an event as **urgent**. These APIs are how you tell it otherwise.

| | You (your code) | Fiber (internal) |
|---|---|---|
| Role | **Mark** updates as non-urgent | **Schedule** them: low-priority lane, render in pieces, interrupt, throw away outdated renders |
| Tools | `startTransition` / `useTransition` (wrap the **state update**), `useDeferredValue` (wrap the **value**, when you don't own the `setState`) | Lanes, the interruptible work loop, the scheduler, `current` / `workInProgress` trees |

```jsx
const [isPending, startTransition] = useTransition();
const handleChange = (e) => {
  setQuery(e.target.value);                              // urgent: the input updates immediately
  startTransition(() => setResults(filter(e.target.value))); // non-urgent: can be interrupted
};

const deferredQuery = useDeferredValue(query);   // lags behind `query` while React is busy
<SlowList query={deferredQuery} />               // memoise SlowList so it's actually skipped
```

**What Fiber does with a transition:** it puts the update in a low-priority lane and renders it fiber by fiber, yielding to the browser between pieces. When a keystroke (high priority) arrives, it pauses or abandons the half-finished render, handles the keystroke, and then restarts the transition with the latest value. Thanks to double buffering, the half-built tree is never shown. (Details in [8.2](#82-usetransition--starttransition) and [8.3](#83-usedeferredvalue).)

**One-line answer:** *"`startTransition`, `useTransition` and `useDeferredValue` are public APIs that developers call to mark updates as non-urgent. Fiber is the internal engine that makes them work: it gives those updates a low-priority lane and renders them in interruptible pieces, so urgent updates like typing can cut in."*

---

### 2.6 Batching (Automatic Batching in React 18)

**Explanation**
**Batching** means React **collects several state updates** and does **one re-render** for all of them, instead of one render per `setState`.

```jsx
function handleClick() {
  setCount(c => c + 1);   // queued
  setFlag(f => !f);       // queued
  setName('Krishna');     // queued
  // → React re-renders ONCE, with all 3 changes
}
```
**Analogy:** a waiter takes **the whole table's order** and then goes to the kitchen once, instead of running to the kitchen after every dish someone orders.

**Why batch?**
1. **Performance:** 3 updates mean 1 render + 1 DOM commit, instead of 3 renders + 3 commits.
2. **Consistency:** the UI never shows a **half-updated state**, e.g. `count` updated but `flag` not yet.

**How it works (simplified)**
```
setCount(...)  → added to the update queue, no render yet
setFlag(...)   → added to the update queue, no render yet
               → your function finishes
React          → processes the whole queue → ONE render → ONE commit
```
`setState` **doesn't re-render immediately**. It schedules an update, and React applies everything in the queue together when the current code finishes.

**React 17 vs React 18**

| Where the updates happen | React 17 | React 18 (`createRoot`) |
|---|---|---|
| React event handler (`onClick`) | ✅ Batched (1 render) | ✅ Batched |
| `setTimeout` / `setInterval` | ❌ Not batched (1 render per update) | ✅ Batched |
| Promise `.then` / after `await` | ❌ Not batched | ✅ Batched |
| Native `addEventListener` handlers | ❌ Not batched | ✅ Batched |

```jsx
fetch('/api/user').then(data => {
  setUser(data);
  setLoading(false);
  // React 17: 2 renders (and for a moment, user is set but loading is still true)
  // React 18: 1 render
});
```
- **Why did React 17 only batch event handlers?** It batched by setting an "I'm inside a React event" flag. Code in `setTimeout` or `.then` runs **later**, after that flag has been switched off. React 18 batches **by time** instead: any updates queued in the same tick are processed together, wherever they come from.
- Automatic batching everywhere only applies with **`createRoot`**. The legacy `ReactDOM.render` keeps the React 17 behaviour.

**A related trap: updater function vs value**
```jsx
// ❌ Ends up +1, not +3: `count` is the same constant in all three calls
setCount(count + 1);   // 0 → 1
setCount(count + 1);   // count is STILL 0 → 1
setCount(count + 1);   // → 1

// ✅ Ends up +3: each updater receives the latest queued value
setCount(c => c + 1);  // 0 → 1
setCount(c => c + 1);  // 1 → 2
setCount(c => c + 1);  // 2 → 3
```
Use the **updater form** whenever the new state depends on the previous state.

**Opting out with `flushSync` (rare)**
Use it when you need the DOM updated **straight away**, for example to measure it or scroll to a newly added item:
```jsx
import { flushSync } from 'react-dom';

const addMessage = (msg) => {
  flushSync(() => {
    setMessages(m => [...m, msg]);   // render + commit happen right here
  });
  listRef.current.lastChild.scrollIntoView();  // the new <li> already exists in the DOM
};
```
Without `flushSync`, the new `<li>` wouldn't be in the DOM yet when `scrollIntoView` runs. ⚠️ It forces a synchronous render and hurts performance, so use it only when you really need it.

**Scenario:** *"After `setCount(count + 1)`, `console.log(count)` shows the old value."*
- `setState` only **queues** the update. Nothing has re-rendered yet.
- `count` is a **constant** in this render's closure. It never changes inside this function call.
- The new value shows up in the **next render**.

Fix: compute the value first and use the variable: `const next = count + 1; setCount(next); sendToAnalytics(next);`. To react to the change *after* the render, use `useEffect(() => ..., [count])`.

**More scenarios**
- *"Three `setCount(count + 1)` calls only add 1."* All three read the same `count` from this render. Use `setCount(c => c + 1)`.
- *"Our app re-renders twice after a fetch in React 17."* No batching outside event handlers. Upgrading to React 18 with `createRoot` fixes it, or wrap the updates in `unstable_batchedUpdates` in React 17.
- *"I need to scroll to the new item right after adding it."* Use `flushSync`, or do the scroll in a `useLayoutEffect`.

**One-line answer:** *"Batching groups several state updates into one re-render, for performance and so the UI never shows a half-updated state. React 17 batched only inside React event handlers, while React 18 with `createRoot` batches automatically everywhere, including timeouts, promises and native listeners. `setState` only queues an update, so the variable keeps its old value until the next render. Use the updater form for sequential updates, and `flushSync` to opt out when you need the DOM updated immediately."*

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

### 2.8 Interview Questions

**Q1. What is the Virtual DOM, and does it make React fast?** ⭐
It's a lightweight description of the UI as JS objects. React compares the new description with the previous one and applies the smallest set of real DOM changes. It isn't faster than perfectly written manual DOM updates. Its benefit is giving you a simple declarative model with good-enough performance automatically.

**Q2. How does reconciliation work?** ⭐
React compares trees using two heuristics that make diffing O(n). Elements of a different type replace the whole subtree. Elements of the same type are updated in place, and keys match children within lists. That's why changing a component's type or position resets its state.

**Q3. What causes a component to re-render?** ⭐
Its state changes, its parent re-renders, or a Context it reads changes. A prop change isn't a separate cause: props change because the parent rendered. By default, a parent's re-render re-renders all its children, unless a child is memoised and its props are unchanged.

**Q4. What's the difference between the render phase and the commit phase?** ⭐
The render phase calls components and works out what changed. It must be pure, and it can be paused or discarded. The commit phase applies changes to the DOM, runs `useLayoutEffect`, and is synchronous. `useEffect` runs afterwards, usually after paint.

**Q5. What is React Fiber?**
It's React's reconciliation engine since version 16. Each component has a fiber node, a unit of work that React can pause, resume or drop. That makes priorities and concurrent rendering possible, instead of one uninterruptible recursive render.

**Q6. What is automatic batching?** ⭐
React groups several state updates into one re-render. Before React 18, this only happened inside React event handlers. Since 18, it also happens in promises, timeouts and native handlers. `flushSync` opts out when you need the DOM updated immediately.

**Q7. Why do my Effects run twice in development?**
StrictMode mounts, unmounts and remounts components in development to reveal missing cleanup. It doesn't happen in production. If double-running causes a bug, the Effect needs a proper cleanup function.

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

A **race condition** happens when several requests are in flight and they **come back in a different order** than they were sent, so a slow, outdated response **overwrites** a newer one. The network never guarantees response order: server load, caching and retries all change the timing.

```jsx
// ❌ Buggy
useEffect(() => {
  fetch(`/api/users/${id}`).then(r => r.json()).then(data => setUser(data));
}, [id]);
```
```
t=0ms    click Profile 1 → request A sent (slow: 2 s)
t=100ms  click Profile 2 → request B sent (fast: 300 ms)
t=400ms  B returns → setUser(profile 2) ✅
t=2000ms A returns → setUser(profile 1) ❌ the UI says "Profile 2" but shows profile 1
```
Real-world examples: search-as-you-type ("re" results arriving after "react" results), switching tabs or filters quickly, pagination.

**Fix 1: `ignore` flag (skip stale responses)**
```jsx
useEffect(() => {
  let ignore = false;                       // belongs to THIS effect run
  fetch(`/api/users/${id}`)
    .then(r => r.json())
    .then(data => { if (!ignore) setUser(data); });
  return () => { ignore = true; };          // a stale response is ignored
}, [id]);
```
Each effect run has **its own** `ignore` (a closure). When `id` changes, React runs the **previous effect's cleanup first**, which sets *that* run's `ignore = true`. When the old response arrives, it sees `true` and does nothing.
```
id=1: ignore₁ = false, request A sent
id=2: cleanup → ignore₁ = true; new run: ignore₂ = false, request B sent
B returns → ignore₂ false → setUser(B) ✅
A returns → ignore₁ true  → skipped ✅
```
⚠️ The old request still completes over the network. Its result is just thrown away.

**Fix 2: `AbortController` (cancel the old request)**
```jsx
useEffect(() => {
  const ctrl = new AbortController();
  fetch(`/api/users/${id}`, { signal: ctrl.signal }).then(r => r.json()).then(setUser)
    .catch(e => { if (e.name !== 'AbortError') setError(e); });   // aborting is not an error
  return () => ctrl.abort();
}, [id]);
```
`ctrl.abort()` **cancels the request in the browser**: the promise rejects with `AbortError`, so `setUser` never runs for the outdated request. The `catch` ignores `AbortError` so you don't show an error for a cancellation you caused yourself.

| | `ignore` flag | `AbortController` |
|---|---|---|
| Prevents stale data | ✅ | ✅ |
| Cancels the network request (saves bandwidth and server work) | ❌ | ✅ |
| Works with any async code | ✅ | Only APIs that accept a `signal` (fetch, axios) |

Both also stop **setState after unmount**, because the cleanup also runs on unmount.

**Fix 3 (production answer): React Query / SWR**
```jsx
const { data: user, isLoading, error } = useQuery({
  queryKey: ['user', id],                   // each id has its own cache entry
  queryFn: ({ signal }) => fetch(`/api/users/${id}`, { signal }).then(r => r.json()),
});
```
Results are stored **per key**, so a late response for `['user', 1]` can never overwrite `['user', 2]`. You also get caching, de-duplication, retries and automatic cancellation through `signal` (see [6.5](#65-react-query--swr-server-state)).

**Search-as-you-type:** combine **debouncing** (fewer requests) with **AbortController** (cancel the ones still in flight). Debouncing alone doesn't prevent races, because two requests can still overlap.

**Scenario:** *"User clicks profile 1 then quickly profile 2, but sees profile 1's data."*
Request 1 came back after request 2 and overwrote it. Fix it in the effect's cleanup with the `ignore` flag or an `AbortController` (ignoring the `AbortError`). In production, use React Query / SWR, which key results per request and cancel outdated ones automatically.

**One-line answer:** *"A fetch race condition happens when responses arrive out of order and an outdated one overwrites newer state. In `useEffect`, the cleanup prevents it: an `ignore` flag discards stale responses, or an `AbortController` cancels the previous request. React Query / SWR solve it by keying results per request."*

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
// 1. Create the context. Default `null` so a missing Provider can be detected.
const ThemeContext = createContext(null);

// 2. Provider component: owns the state and exposes a memoised value
function ThemeProvider({ children }) {
  const [theme, setTheme] = useState('light');
  const toggleTheme = useCallback(
    () => setTheme(t => (t === 'light' ? 'dark' : 'light')),
    []
  );
  const value = useMemo(() => ({ theme, toggleTheme }), [theme, toggleTheme]);   // stable reference
  // React 19: you can render ThemeContext directly instead of ThemeContext.Provider
  return (
    <ThemeContext.Provider value={value}>
      {children}
    </ThemeContext.Provider>
  );
}

// 3. Custom hook with a guard: consumers never touch the raw context
function useTheme() {
  const ctx = useContext(ThemeContext);
  if (!ctx) throw new Error('useTheme must be used inside <ThemeProvider>');
  return ctx;
}

// 4. Usage: Layout passes no theme props, but Header still gets the theme
function App() {
  return (
    <ThemeProvider>
      <Layout />
    </ThemeProvider>
  );
}

function Layout() {
  return <main><Header /></main>;   // knows nothing about theme
}

function Header() {
  const { theme, toggleTheme } = useTheme();
  return (
    <header className={'header header--' + theme}>
      <button onClick={toggleTheme}>
        Switch to {theme === 'light' ? 'dark' : 'light'}
      </button>
    </header>
  );
}
```

Why this shape:
- **`createContext(null)`** makes the guard in `useTheme` work. With a default like `'light'`, a component outside the Provider would silently get the wrong value.
- **Provider as its own component taking `children`**: when the theme changes, `ThemeProvider` re-renders, but `children` (`<Layout />`) is the same element created by `App`, so `Layout` is skipped. Only components that call `useTheme` re-render.
- **`useMemo` + `useCallback`** keep `value` the same object between renders unless `theme` actually changes.
- **Expose `toggleTheme`, not `setTheme`**: consumers get an intent-based API and can't set an invalid theme.

**Performance pitfall:** **every consumer re-renders** when the Provider's `value` changes (by reference). `React.memo` doesn't stop it.

**Scenario:** *"The whole app re-renders on every keystroke in a form whose state lives in Context."*
Fixes:
1. **Split contexts** by how often they change (e.g. `UserContext` vs `FormContext`).
2. **Separate state and dispatch** into two contexts. Components that only dispatch won't re-render.
3. **Memoise the `value`** object.
4. Use a **store with selectors** (Zustand, Redux) for state that changes often.

```jsx
// Fix 2: separate state and dispatch
const CountStateContext = createContext(null);
const CountDispatchContext = createContext(null);

function CountProvider({ children }) {
  const [count, dispatch] = useReducer((c, action) => (action === 'inc' ? c + 1 : c - 1), 0);
  return (
    <CountDispatchContext.Provider value={dispatch}>   {/* dispatch never changes */}
      <CountStateContext.Provider value={count}>
        {children}
      </CountStateContext.Provider>
    </CountDispatchContext.Provider>
  );
}

function CountDisplay() {
  const count = useContext(CountStateContext);       // re-renders when count changes
  return <p>{count}</p>;
}

function IncrementButton() {
  const dispatch = useContext(CountDispatchContext); // never re-renders on count change
  return <button onClick={() => dispatch('inc')}>+1</button>;
}
```

---

### 3.8 `useMemo` & `useCallback`

**Explanation**
- `useMemo(() => compute(a, b), [a, b])` caches a **computed value**.
- `useCallback(fn, [deps])` caches a **function reference**. It's the same as `useMemo(() => fn, deps)`.

They help in only **two cases**:
1. **Expensive calculations** (filtering/sorting thousands of items).
2. **Referential stability**: the value is passed to a `React.memo` child, or used as an effect/hook dependency.

**useMemo vs useCallback**
| | `useMemo` | `useCallback` |
|---|---|---|
| Caches | The **result** of a function | The **function itself** |
| Returns | A value (number, array, object…) | A function |
| Runs your function? | Yes, during render, when deps change | No. You call it later (event, effect, child) |
| Syntax | `useMemo(() => value, deps)` | `useCallback(fn, deps)` |
| Without it | The calculation re-runs every render | A new function object is created every render |
| Problem it solves | Repeating slow work | A new reference breaking `React.memo` or re-running effects |
| Typical use | Filtered/sorted lists, derived totals, stable objects (e.g. a context `value`) | Handlers passed to memoised children, functions used in `useEffect` deps |
| Pointless when | The calculation is cheap and the result isn't passed to anything memoised | The child isn't memoised and the function isn't a dependency |
| Relationship | General tool | Shortcut: `useCallback(fn, deps)` = `useMemo(() => fn, deps)` |

#### `useMemo` example 1: skip an expensive calculation
```jsx
function UserTable({ users, sortBy }) {        // users: 10,000 rows
  const [darkMode, setDarkMode] = useState(false);

  // ✅ Sorts only when users or sortBy change
  const sorted = useMemo(
    () => [...users].sort((a, b) => a[sortBy].localeCompare(b[sortBy])),
    [users, sortBy]
  );
  // ❌ Without useMemo, toggling dark mode would re-sort 10,000 rows

  return (
    <div className={darkMode ? 'dark' : 'light'}>
      <button onClick={() => setDarkMode(d => !d)}>Toggle theme</button>
      <ul>{sorted.map(u => <li key={u.id}>{u.name}</li>)}</ul>
    </div>
  );
}
```

#### `useMemo` example 2: keep an object stable for an effect
```jsx
function ChatRoom({ roomId, serverUrl }) {
  const [message, setMessage] = useState('');

  // ✅ Same object until roomId or serverUrl change
  const options = useMemo(() => ({ roomId, serverUrl }), [roomId, serverUrl]);

  useEffect(() => {
    const conn = createConnection(options);
    conn.connect();
    return () => conn.disconnect();
  }, [options]);
  // ❌ Without useMemo: a new object every render, so typing a message reconnects the chat

  return <input value={message} onChange={e => setMessage(e.target.value)} />;
}
```

#### `useCallback` example 1: stable handler for a memoised child
```jsx
function TodoApp() {
  const [todos, setTodos] = useState([]);
  const [text, setText] = useState('');

  // ✅ Same function every render, so TodoList skips re-rendering while you type
  const handleDelete = useCallback(id => {
    setTodos(prev => prev.filter(t => t.id !== id));
  }, []);   // no deps needed thanks to the functional updater
  // ❌ Without useCallback: a new function every keystroke, so React.memo sees a changed prop

  return (
    <>
      <input value={text} onChange={e => setText(e.target.value)} />
      <TodoList todos={todos} onDelete={handleDelete} />
    </>
  );
}

const TodoList = React.memo(function TodoList({ todos, onDelete }) {
  return todos.map(t => <li key={t.id} onClick={() => onDelete(t.id)}>{t.title}</li>);
});
```

#### `useCallback` example 2: function used as an effect dependency
```jsx
function SearchResults({ query }) {
  const [results, setResults] = useState([]);

  // ✅ New function only when query changes
  const fetchResults = useCallback(async () => {
    const res = await fetch('/api/search?q=' + encodeURIComponent(query));
    setResults(await res.json());
  }, [query]);

  useEffect(() => { fetchResults(); }, [fetchResults]);
  // ❌ Without useCallback: a new function every render, so the effect fetches after every render

  return <ul>{results.map(r => <li key={r.id}>{r.title}</li>)}</ul>;
}
```

#### Same thing, two ways (and a common mistake)
```jsx
const a = useCallback(() => console.log('hi'), []);
const b = useMemo(() => () => console.log('hi'), []);   // identical to a

const c = useMemo(() => console.log('hi'), []);         // ❌ runs once and caches undefined (the result), not the function
```

**Memory trick:** `useMemo` remembers the **answer**, `useCallback` remembers the **question** (the function).

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

**Scenario:** *"A label and its input get mismatched IDs between server and client."* A counter or `Math.random()` gives different values on each side. `useId` generates a stable ID that matches across SSR and hydration.

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

### 3.13 Interview Questions

**Q1. What's the difference between `useEffect` and `useLayoutEffect`?** ⭐
`useEffect` usually runs after the browser paints, so it doesn't block rendering. `useLayoutEffect` runs synchronously after DOM changes but before paint, so it's used to measure layout and adjust it without a visible flicker. Default to `useEffect`, because `useLayoutEffect` blocks painting.

**Q2. How does the `useEffect` dependency array work, and when does cleanup run?** ⭐
No array runs after every render, `[]` runs after mount, and `[a, b]` runs when a value changes by `Object.is`. Cleanup runs before the Effect runs again and on unmount. Objects and functions created during render change every time, which is a common cause of loops.

**Q3. `useMemo` vs `useCallback`: when should you use them?** ⭐
`useMemo` caches a calculated value, and `useCallback` caches a function. Use them for genuinely expensive calculations, or to keep a reference stable for a memoised child or a dependency array. Using them everywhere adds cost and noise, and the React Compiler removes most of the need.

**Q4. `useRef` vs `useState`?** ⭐
Changing state triggers a re-render, while changing `ref.current` doesn't. Use state for anything shown in the UI. Use refs for DOM nodes and mutable values such as timer IDs. Don't read or write refs during render, because the output would no longer be pure.

**Q5. What are the Rules of Hooks, and why do they exist?** ⭐
Call Hooks only at the top level, and only from components or custom Hooks. React identifies Hooks by their **call order** on each render, so a Hook inside a condition would shift the order and connect state to the wrong Hook.

**Q6. What is a stale closure in React?** ⭐
A function keeps the values from the render in which it was created. A `setInterval` in a mount-only Effect, for example, always sees the first `count`. Fix it with a functional update (`setCount(c => c + 1)`), correct dependencies, a ref holding the latest value, or `useEffectEvent`.

**Q7. When would you use `useReducer` instead of `useState`?**
When several state values change together, the next state depends on complex logic, or you want updates described as named actions. Reducers are also easy to test. Stay with `useState` for simple independent values.

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

**Scenario:** *"Calling `this.setState({ count: this.state.count + 1 })` three times only adds 1."* All three read the same batched value. Use `this.setState(prev => ({ count: prev.count + 1 }))`, which receives the latest state each time.

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

**`PureComponent` vs `React.memo`**

| | `PureComponent` | `React.memo` |
|---|---|---|
| Used with | Class components | Function components |
| Compares | **Props and state** (shallow) | **Props only** (shallow) |
| Custom comparison | Override `shouldComponentUpdate` (returns `true` to **render**) | 2nd argument `arePropsEqual` (returns `true` to **skip**) |
| State updates | Skipped if the state is shallowly equal | Always re-render, but `useState` bails out when the new value is the same (`Object.is`) |

Note that the two custom-comparison functions return opposite things, which interviewers like to ask about. In new code, the React Compiler handles memoisation automatically (see 7.11).

---

### 4.4 Error Boundaries (still class-only)

See [Section 13.1](#131-error-boundaries) for full details. In short: a class with `static getDerivedStateFromError` (to render a fallback) and/or `componentDidCatch` (to log). There's no hook equivalent; use the `react-error-boundary` package to wrap one.

| Method | Runs during | Use for |
|---|---|---|
| `static getDerivedStateFromError(error)` | Render phase (must be pure) | Returning state that shows the fallback UI |
| `componentDidCatch(error, info)` | Commit phase (side effects allowed) | Logging, with `info.componentStack` showing where it failed |

```jsx
class ErrorBoundary extends React.Component {
  state = { hasError: false };

  static getDerivedStateFromError() {
    return { hasError: true };
  }

  componentDidCatch(error, info) {
    logError(error, info.componentStack);
  }

  render() {
    return this.state.hasError ? this.props.fallback : this.props.children;
  }
}
```

**Scenario:** *"A legacy widget crashes and blanks the whole page."* Wrap it in a boundary with a small fallback, so the rest of the page stays usable, and log the error from `componentDidCatch`.

---

### 4.5 Interview Questions

**Q1. How do class lifecycle methods map to Hooks?** ⭐
`componentDidMount` maps to `useEffect` with `[]`, `componentDidUpdate` to `useEffect` with dependencies, and `componentWillUnmount` to the Effect's cleanup. The mapping isn't exact, though: Effects are better thought of as synchronising with something, not as lifecycle events.

**Q2. Why did React move from classes to Hooks?** ⭐
Hooks reuse stateful logic without HOC or render-prop wrappers, group related code together instead of splitting it across lifecycle methods, and avoid `this` binding problems. Classes still work, and error boundaries still need them.

**Q3. Is `setState` synchronous?** ⭐
It's treated as asynchronous, because React batches updates. Reading `this.state` straight after calling it can give the old value. Use the functional form, `setState(prev => ...)`, when the next value depends on the previous one, and the second callback argument to run code after the update.

**Q4. What's the difference between `Component` and `PureComponent`?**
`PureComponent` implements `shouldComponentUpdate` with a shallow comparison of props and state, so it skips re-rendering when they're unchanged. Mutating an object in place won't be detected. `React.memo` is the function-component equivalent.

**Q5. Why are error boundaries still class components?**
They rely on `getDerivedStateFromError` and `componentDidCatch`, and React has no Hook equivalent. Most teams use the `react-error-boundary` package, which wraps that class in a simple component.

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

**Scenario:** *"Consumers of our `<Toggle>` want it to work both with and without their own state."* Support both modes: use the `value` prop when it's provided, and internal state otherwise. Prop getters like `getToggleProps()` return the correct handlers and ARIA attributes, and merge in the consumer's handlers.

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

### 5.9 Interview Questions

**Q1. HOCs vs render props vs custom Hooks?** ⭐
All three share logic. HOCs wrap a component and can cause wrapper nesting and prop collisions. Render props pass a function but can make JSX deeply nested. Custom Hooks are now the default because they share logic with no extra components. HOCs are still useful for cross-cutting wrappers such as auth checks.

**Q2. What are compound components?** ⭐
A group of components that work together and share implicit state through Context, like `<Tabs>`, `<Tabs.List>` and `<Tabs.Panel>`. Users control the structure with plain JSX instead of a long list of configuration props. Component libraries use this pattern widely.

**Q3. What are Portals, and when would you use them?** ⭐
`createPortal` renders children into a different DOM node, usually `document.body`. Use it for modals, tooltips and dropdowns, which would otherwise be clipped by `overflow: hidden` or trapped in a stacking context. Events still bubble through the React tree, not the DOM tree.

**Q4. How do you pass a ref to a child component?** ⭐
In React 19, `ref` is an ordinary prop on function components. Before that, you wrapped the component in `forwardRef`. Use `useImperativeHandle` to expose only specific methods instead of the whole DOM node.

**Q5. Why does React prefer composition over inheritance?**
Passing components through `children` or props gives you all the flexibility you need without tight parent-child coupling. Specialised components just render a general one with preset props. Component inheritance is almost never used in React.

**Q6. Is the container/presentational pattern still relevant?**
The idea of separating logic from UI still is, but custom Hooks usually replace container components. It's useful for keeping UI components simple to test and show in Storybook.

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

### 6.7 Interview Questions

**Q1. Context vs Redux: when do you use each?** ⭐
Context passes values down the tree and isn't really a state manager. It suits low-frequency data like theme, locale and the current user. Redux Toolkit suits complex, frequently updated shared client state, with selectors, middleware and DevTools. Many apps need neither once server data moves to React Query.

**Q2. Why can Context cause performance problems?** ⭐
Every consumer re-renders when the Provider's value changes, even if it only uses part of that value. Fix it by splitting contexts, memoising the value, separating state from dispatch, or using a store with selectors.

**Q3. What's the difference between client state and server state?** ⭐
Client state belongs to the browser, like a modal being open. Server state lives on a server and is only a cached copy here, so it can go stale. It needs caching, refetching and de-duplication, which is why React Query or SWR is a better fit than Redux for it.

**Q4. Explain the Redux data flow.** ⭐
A component dispatches an action, the reducer calculates the next state from the current state and the action, and the store updates. Subscribed components then re-render if the data they selected changed. Redux Toolkit reduces the boilerplate with `createSlice`, and Immer lets you write "mutating" reducer code safely.

**Q5. How does React Query work at a high level?** ⭐
It caches data by query key. It returns cached data immediately, refetches stale data in the background, de-duplicates identical requests, and handles retries. `staleTime` controls freshness, `gcTime` controls how long unused data stays, and mutations invalidate related keys.

**Q6. How do you decide where state should live?**
Keep it as local as possible. URL for shareable state, React Query for server data, form libraries for form state, local state for single components, and Context or a store only for genuinely shared client state.

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

**Match the symptom to the likely cause**

| Symptom | Likely cause | Tool | First fixes |
|---|---|---|---|
| Blank page for a long time | Large JS bundle, CSR only | Network tab, bundle analyser | Code splitting, SSR, remove heavy dependencies |
| Main image appears late (poor LCP) | Unoptimised or late-discovered image | Lighthouse, Performance tab | Preload it, modern formats, `fetchpriority="high"` |
| Clicks or typing lag (poor INP) | Too many re-renders, long tasks | React Profiler, Performance tab | Move state down, `startTransition`, virtualise |
| Page jumps while loading (poor CLS) | Missing sizes, late content | Layout Shift regions | Set dimensions, use skeletons |
| Scrolling stutters | Too many DOM nodes, layout work | Performance tab | Virtualise, animate with `transform` |
| Gets slower over time | Memory leak | Memory tab | See [Section 26](#26-memory-leaks) |

**Don't optimise blindly.** Each `useMemo` or `React.memo` adds code and comparison work, so add them only where a profile shows a real benefit. Test with CPU throttling too, because your laptop is much faster than a typical user's phone.

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

**Props only:** `memo` compares **props**, not state. A component's own state change or a Context change always re-renders it. Setting state to the **same** value (`Object.is`) bails out on its own, with or without `memo`. (Class `PureComponent` compares props **and** state, see 4.3.)

**With the React Compiler:** the compiler adds the equivalent of `memo`, `useMemo` and `useCallback` automatically at build time, so new code rarely needs a manual `React.memo`. It's still useful in codebases without the compiler, or as an explicit escape hatch (see [7.11](#711-react-compiler)).

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

**Scenario:** *"Typing in a search box re-renders a large chart on the same page."* The input's state lives in a shared parent. Move it into a small `<SearchBox>` component, so the chart isn't part of the re-render. No memoisation is needed.

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

### 7.12 Interview Questions

**Q1. How do you approach a slow React app?** ⭐
Measure first. Separate slow initial load (bundle, network, images) from slow interactions (re-renders, long tasks). Profile with React DevTools and the Chrome Performance tab, fix the largest cause with the simplest change, and measure again.

**Q2. When doesn't `React.memo` help?** ⭐
When props change on every render, such as inline objects and functions without stable references. It also adds little for cheap components, or when `children` is passed, because JSX creates a new object each time. Moving state down or passing components as `children` often works better.

**Q3. How do you reduce unnecessary re-renders?** ⭐
Move state closer to where it's used, pass expensive subtrees as `children`, split Contexts, use selectors, then memoise where the profiler shows a real benefit. Also avoid changing keys, which remounts components.

**Q4. How do you implement code splitting?** ⭐
Use `React.lazy` with dynamic `import()` and wrap it in `<Suspense>`. Split by route first, then large rarely used components such as charts and editors. Prefetch likely next routes so splitting doesn't add a delay.

**Q5. How do you render a list of 10,000 rows?** ⭐
Virtualise it with TanStack Virtual or react-window, so only visible rows exist in the DOM. Pagination is often simpler when users don't need continuous scrolling. Also check whether the backend should return that much at all.

**Q6. What are the Core Web Vitals?** ⭐
LCP measures load speed (good is 2.5 s or less), INP measures responsiveness (200 ms or less), and CLS measures visual stability (0.1 or less). Use field data from real users, because lab tests don't show the full picture.

**Q7. Does the React Compiler remove the need for `useMemo`?**
For most new code, yes. It adds memoisation automatically at build time, as long as your code follows the Rules of React. Manual memoisation remains an escape hatch, and existing calls are harmless to leave in place.

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

**Scenario:** *"A search box lags as it filters a big list passed in from a parent."* You don't own the setter, so use `const deferred = useDeferredValue(query)` and pass it to a memoised list. The input updates at once, and the list follows when React has time.

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

### 8.6 Interview Questions

**Q1. What is concurrent rendering?** ⭐
React can prepare an update in the background, pause it for something more urgent, and discard it if it's out of date. It isn't multi-threading: it's interruptible rendering on the main thread. You use it through features such as `useTransition`, `useDeferredValue` and Suspense.

**Q2. `useTransition` vs `useDeferredValue`?** ⭐
`useTransition` wraps the state update you control and provides `isPending`. `useDeferredValue` wraps a **value** you receive, such as a prop, and gives a lagging copy. Use the first when you own the setter, and the second when you don't.

**Q3. `startTransition` vs debouncing?** ⭐
Debouncing delays the work by a fixed time, which reduces network calls. `startTransition` starts immediately but lets urgent updates interrupt rendering. They solve different problems: debounce for API calls, and transitions for expensive renders.

**Q4. What is Suspense?** ⭐
A boundary that shows a fallback while something inside isn't ready. It works with `React.lazy`, Suspense-enabled data libraries, `use()`, and Server Components. It doesn't detect ordinary `useEffect` fetches.

**Q5. What is streaming SSR?**
The server sends HTML in pieces. The page shell arrives first with fallbacks, and each Suspense boundary streams in when its data is ready. Users see content sooner, and selective hydration makes the parts they interact with active first.

**Q6. What is tearing?**
Different components showing different values from the same external store during one concurrent render. `useSyncExternalStore` prevents it, which is why state libraries use it.

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

**Scenario:** *"`use(fetch(...))` inside a component loops forever."* Each render creates a new Promise, which suspends again. Create the Promise outside render, such as in a Server Component or a cache, and pass it down as a prop.

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

### 9.6 React 19.2: `<Activity>` & `useEffectEvent`

**Explanation**
React 19.2 (October 2025) added two APIs that are now common interview topics.

**`<Activity>`** hides and restores part of the UI **without losing its state**. It's a third option between "keep it mounted" and "unmount it".

| | `{show && <Panel />}` | CSS `display: none` | `<Activity mode="hidden">` |
|---|---|---|---|
| State kept | ❌ Lost | ✅ | ✅ |
| DOM kept | ❌ | ✅ | ✅ |
| Effects while hidden | Removed | **Still running** | **Cleaned up**, re-created when visible |
| Updates while hidden | None | Normal priority | Lower priority |

```jsx
<Activity mode={tab === 'chat' ? 'visible' : 'hidden'}>
  <ChatPanel />          {/* draft text and scroll position survive a tab switch */}
</Activity>

<Activity mode="hidden">
  <SettingsPage />       {/* pre-render at low priority so it opens instantly */}
</Activity>
```

Things to mention:
- **Pre-rendering** only fetches data through Suspense-enabled sources (such as `use`). Data fetched in an Effect isn't loaded, because Effects don't run while hidden.
- **The DOM stays**, so a hidden `<video>` keeps playing unless a cleanup Effect pauses it.
- `<Activity>` boundaries also allow **selective hydration**, even when nothing is hidden.

**`useEffectEvent`** extracts non-reactive logic from an Effect. The function always sees the latest props and state, but it doesn't make the Effect re-run.

```jsx
function ChatRoom({ roomId, theme }) {
  const onConnected = useEffectEvent(() => {
    showToast('Connected!', theme);        // always the latest theme
  });

  useEffect(() => {
    const conn = createConnection(roomId);
    conn.on('connected', () => onConnected());
    conn.connect();
    return () => conn.disconnect();
  }, [roomId]);                            // theme isn't a dependency: no reconnect on theme change
}
```

Rules: only call it from inside Effects, don't pass it to other components, and leave it out of the dependency array. It fixes the stale-closure trade-off from [3.11](#311-stale-closures) without silencing the lint rule.

**Also in 19.2:** `cacheSignal` (for RSC), Performance Tracks in Chrome DevTools, and partial pre-rendering APIs.

**Scenario:** *"Users lose their half-written message when they switch tabs and come back."*
Conditional rendering unmounts the tab, so its state is destroyed. Wrap each tab in `<Activity>`. State and DOM survive, and Effects such as polling or subscriptions stop while the tab is hidden. Lifting the state up would also work, but it re-renders the parent on every keystroke.

---

### 9.7 React 19.3: View Transitions, Fragment Refs & `browser()`

**Explanation**
React 19.3 (September 2026) made two long-awaited APIs stable and added a few smaller ones.

**`<ViewTransition>`** animates elements with the browser's View Transition API. It only animates updates marked as **Transitions**: `startTransition`, a `<Suspense>` reveal, or `useDeferredValue`. Urgent updates aren't animated.

```jsx
import { ViewTransition, addTransitionType, startTransition } from 'react';

function nextSlide() {
  startTransition(() => {
    addTransitionType('next');             // records WHY the transition happened
    setSlide(s => s + 1);
  });
}

<ViewTransition
  enter={{ next: 'from-right', previous: 'from-left' }}
  exit={{ next: 'to-left', previous: 'to-right' }}
>
  <Slide />
</ViewTransition>
```

| Trigger | When it fires |
|---|---|
| **enter** | The `<ViewTransition>` is added |
| **exit** | The `<ViewTransition>` is removed |
| **update** | Its children change style or content |
| **share** | A named `<ViewTransition>` is removed in one place and added in another (shared-element animation) |

The default animation is a cross-fade. Enter and exit only apply if the `<ViewTransition>` comes before any DOM node in its subtree, and each shared `name` must be unique at any given time. It currently works in the DOM only.

**Fragment refs** act on a group of children **without a wrapper element**. The ref gives a `FragmentInstance` with methods for events, focus, observers and measurement:

```jsx
function Results({ items }) {
  const ref = useRef(null);
  useEffect(() => { ref.current.focus(); }, []);   // focuses the first focusable child

  return (
    <Fragment ref={ref}>
      {items.map(item => <Row key={item.id} {...item} />)}
    </Fragment>
  );
}
```

**`browser()`** opts a component out of server rendering, replacing the old `mounted` flag pattern:

```jsx
import { use } from 'react';
import { browser } from 'react-dom';

function TimeZone() {
  use(browser());       // on the server this suspends, so the nearest fallback goes into the HTML
  return <p>{Intl.DateTimeFormat().resolvedOptions().timeZone}</p>;
}
```

**Other changes**
- Server Components can render a Context imported from a `'use client'` module directly (`<UserContext value={user}>`), so no wrapper Provider is needed. They still can't create one.
- **Trusted Types** support: values like `TrustedHTML` are no longer converted to strings.
- A slow Transition no longer blocks unrelated Transitions.
- Strict Mode now double-invokes Effects during hydration too.
- Calling `use` conditionally logs a warning.

**Scenario:** *"A time-zone label causes a hydration mismatch. How do you fix it?"*
The server and browser produce different text. Previously you'd set a `mounted` flag in an Effect, which renders twice. In 19.3, wrap it in `<Suspense>` and call `use(browser())`: the server sends the fallback and the client fills in the real value.

---

### 9.8 Interview Questions

**Q1. What are the main new features in React 19?** ⭐
Actions and async transitions, `useActionState`, `useFormStatus`, `useOptimistic`, the `use()` API, `ref` as a prop, `<Context>` as a provider, document metadata in components, and stable Server Components and Server Actions. Later releases added `<Activity>`, `useEffectEvent` and `<ViewTransition>`.

**Q2. What are Actions?** ⭐
Async functions used in transitions or passed to `<form action={...}>`. React manages the pending state, errors and optimistic updates, and resets uncontrolled forms on success. That replaces most hand-written `isLoading` and `error` state.

**Q3. How does `useOptimistic` work?** ⭐
It shows a temporary value while an async action runs, then returns to the real state when the action finishes. If the action fails, the optimistic value disappears on its own, so you only need to handle the error message.

**Q4. How is `use()` different from other Hooks?** ⭐
It reads a Promise or Context, and it can be called conditionally and in loops, unlike normal Hooks. With a Promise, it suspends until the value resolves. Create the Promise outside the render, or pass it from a Server Component, so it isn't recreated every time.

**Q5. `<Activity>` vs conditional rendering?** ⭐
Conditional rendering unmounts and loses state. `<Activity mode="hidden">` keeps state and DOM but cleans up Effects and lowers update priority. Use it for tabs and views users are likely to return to.

**Q6. Are Server Actions secure by default?**
No. They're public HTTP endpoints, so anyone can call them directly. Validate input, and check authentication and authorisation inside each action, just as you would in an API route.

**Q7. How would you upgrade from React 18 to 19?**
Move to 18.3 first and fix its deprecation warnings. Run the official codemods, update TypeScript types and third-party libraries, and test on a branch. Then roll out with monitoring.

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

**Caching layers** (often asked; the defaults changed in Next 15 and again in 16; see [10.6](#106-nextjs-15--16-cache-components-proxyts--async-apis)):
- Request memoisation (same `fetch` de-duplicated within one render)
- Data cache (`fetch` cache, `revalidate`, `revalidateTag`)
- Full route cache (static HTML/RSC)
- Router cache (client-side, for back/forward)

**Middleware example** (Next 16 renames this to `proxy.ts`; see [10.6](#106-nextjs-15--16-cache-components-proxyts--async-apis)):
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
  const { id } = await params;       // params is a Promise in Next 15+
  const p = await getProduct(id);
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

### 10.6 Next.js 15 & 16: Cache Components, `proxy.ts` & Async APIs

**Explanation**
Caching is the most-asked Next.js topic, and the defaults have changed twice.

| Version | Caching model |
|---|---|
| **Next 14** | Implicit: `fetch` and GET Route Handlers cached by default |
| **Next 15** | `fetch`, GET Route Handlers and client page navigation **uncached by default** |
| **Next 16** | **Cache Components**: dynamic by default, and caching is an explicit opt-in with `"use cache"` |

**Cache Components** are enabled with `cacheComponents: true` in `next.config.ts`. They build on Partial Prerendering: a page gets a static shell, with dynamic parts streamed through `<Suspense>`.

```tsx
import { cacheLife, cacheTag } from 'next/cache';

async function getProducts() {
  'use cache';                 // cache key generated by the compiler
  cacheLife('hours');
  cacheTag('products');
  return db.products.findMany();
}
```

**Invalidation APIs (Next 16)**

| API | Use it when | Behaviour |
|---|---|---|
| `revalidateTag('products', 'max')` | Content can be briefly stale | Serves cached data and refreshes in the background. The second argument is now required. |
| `updateTag('products')` | The user must see their own change (Server Actions only) | Expires and reads fresh data in the same request (read-your-writes) |
| `refresh()` | Only uncached data needs refreshing (Server Actions only) | Doesn't touch the cache |

**Other changes to know**
- **`proxy.ts` replaces `middleware.ts`**: rename the file and the exported function. It runs on the Node.js runtime. `middleware.ts` still works for Edge use cases but is deprecated.
- **Request APIs are async**: `await params`, `await searchParams`, `await cookies()`, `await headers()`. Synchronous access was removed in 16.
- **Turbopack is the default bundler** (`--webpack` opts out).
- **React Compiler support is stable** (`reactCompiler: true`) but not on by default.
- `next lint` was removed, and `next build` no longer runs the linter.

```tsx
// Next 15+: params is a Promise
export default async function Page({ params }: { params: Promise<{ id: string }> }) {
  const { id } = await params;
  return <Product id={id} />;
}
```

**Scenario:** *"After saving their profile, a user still sees the old name."*
`revalidateTag` is stale-while-revalidate, so the next request can still get the old value. Inside the Server Action use `updateTag('user-42')` for read-your-writes. Keep `revalidateTag` for content like blog posts, where a short delay is fine.

---

### 10.7 Interview Questions

**Q1. CSR vs SSR vs SSG vs ISR?** ⭐
CSR renders in the browser, which is simple but slow to first content. SSR renders on each request, so it suits personal, fresh pages. SSG renders at build time, so it's the fastest but can go stale. ISR is static with background regeneration. Choose per page, not for the whole app.

**Q2. What's the difference between Server Components and SSR?** ⭐
SSR turns components into HTML for the first load, then hydrates them on the client, so their JS is still shipped. Server Components only run on the server, and their code never reaches the browser. They work together: RSC reduces JS, and SSR gives fast first HTML.

**Q3. What is hydration, and what causes a mismatch?** ⭐
Hydration attaches React to server-rendered HTML. A mismatch happens when the first client render differs from the server's HTML, because of `Date.now()`, `Math.random()`, `window` checks, locale formatting or invalid HTML nesting. Render the same output first, then update after mount, or use `use(browser())` in React 19.3.

**Q4. When do you need `'use client'`?** ⭐
When a component uses state, Effects, event handlers or browser APIs. It marks a boundary: that file and its imports go into the client bundle. Keep the boundary as low in the tree as possible, and pass Server Components through `children` to keep them on the server.

**Q5. How does caching work in the latest Next.js?** ⭐
From Next 16 with Cache Components, code is dynamic unless you opt in with `"use cache"`, plus `cacheLife` and `cacheTag`. `revalidateTag` refreshes in the background, while `updateTag` gives read-your-writes inside Server Actions.

**Q6. When would you avoid SSR?**
For apps behind a login where SEO doesn't matter, such as internal dashboards. SSR adds server cost and complexity. A well-optimised SPA is often enough, and you can keep SSR for public pages.

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

### 11.6 React Router v7 & TanStack Router

**Explanation**
**React Router v7** merged Remix into React Router, so there's now one `react-router` package with three additive modes. Import from `react-router`; `react-router-dom` was the older package name.

| Mode | Adds | Choose it when |
|---|---|---|
| **Declarative** | `<BrowserRouter>`, `<Routes>`, navigation hooks | A simple SPA, or your data layer (like React Query) handles loading |
| **Data** | `createBrowserRouter`, loaders, actions, fetchers | You want built-in data loading but control your own bundling |
| **Framework** | A Vite plugin with SSR, code splitting, pre-rendering and type-safe route modules | You want a full framework (this is what Remix was) |

The official advice is to use the least framework you need.

```tsx
// Framework mode: routes/product.tsx
import type { Route } from './+types/product';

export async function loader({ params }: Route.LoaderArgs) {
  return getProduct(params.id);
}

export default function Product({ loaderData }: Route.ComponentProps) {
  return <h1>{loaderData.name}</h1>;       // loaderData is fully typed
}
```

**TanStack Router** is built around **end-to-end type safety**. Route paths, params and search params are all typed, so a broken link is a compile error. Its standout feature is validated search params, which treats the URL as typed state.

```tsx
const productsRoute = createFileRoute('/products')({
  validateSearch: z.object({ page: z.number().catch(1), sort: z.string().optional() }),
  component: Products,
});

function Products() {
  const { page } = productsRoute.useSearch();   // typed as number
}
```

| | React Router v7 | TanStack Router | Next.js App Router |
|---|---|---|---|
| Best at | Mature, flexible, huge adoption | Type safety, typed search params | Full-stack React with RSC |
| Rendering | SPA or SSR by mode | SPA-first (SSR through TanStack Start) | Server-first |
| Data loading | Loaders and actions | Loaders, pairs well with TanStack Query | Server Components, Server Actions |

**Scenario:** *"A dashboard has a dozen filters in the URL, and bugs keep appearing from mistyped or invalid params."*
Plain `useSearchParams` gives you untyped strings. TanStack Router validates search params against a schema, so invalid values fall back to defaults and typos fail at compile time. In an existing React Router app, parse the params through one Zod schema in a shared hook.

---

### 11.7 Interview Questions

**Q1. How does client-side routing work?** ⭐
The router uses the History API to change the URL without a page reload, then renders the matching components. Links are intercepted, and the router listens for back and forward navigation through `popstate`.

**Q2. Why does refreshing a route give a 404 in production?** ⭐
The browser asks the server for `/dashboard`, but only `index.html` exists. Configure the server or CDN to return `index.html` for unknown routes, so the client router can take over. Hash routing avoids this, at the cost of ugly URLs.

**Q3. How do you implement protected routes?** ⭐
Wrap routes in a component or loader that checks auth and redirects to login, keeping the intended URL so you can return the user there. Handle the "still checking" state to avoid a flash of the wrong page. This is only for UX: the server must enforce access to data.

**Q4. Route loaders vs `useEffect` for data?** ⭐
Loaders start fetching as navigation begins, in parallel with the code, which avoids render-then-fetch waterfalls. `useEffect` fetches only after the component has rendered. Loaders also integrate with pending UI and errors at route level.

**Q5. When should state go in the URL?** ⭐
When it should survive a refresh, or be shareable or bookmarkable, such as filters, sort, page and selected tab. Temporary UI like an open tooltip stays in local state.

**Q6. Which router would you choose today?**
Next.js routing for a full-stack React app. React Router v7 for a flexible SPA or a Vite-based framework. TanStack Router when strict type safety and typed search params matter most.

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

**Scenario:** *"A 60-field form lags on every keystroke."* Controlled state re-renders everything for each character. Switch to React Hook Form, so inputs are uncontrolled, and use `useWatch` only in the few places that need live values.

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

### 12.5 Interview Questions

**Q1. Controlled or uncontrolled inputs for forms?** ⭐
Controlled inputs suit forms where the UI reacts to every change, such as live validation or dependent fields. Uncontrolled inputs mean fewer re-renders and less code, so they work well for large or simple forms. React Hook Form uses uncontrolled inputs for this reason.

**Q2. Why is React Hook Form faster than Formik?** ⭐
It registers inputs through refs, so typing doesn't re-render the whole form. Formik traditionally keeps values in React state, which re-renders on every keystroke. The difference is most visible on large forms.

**Q3. How do you handle form validation?** ⭐
Define a schema with Zod or Yup and connect it through a resolver, so one schema gives you validation and TypeScript types. Show errors on blur or submit rather than on every keystroke. Always validate again on the server, because client validation is only for UX.

**Q4. How do you show server-side validation errors?** ⭐
Have the API return errors keyed by field, then map them onto the form with `setError` in React Hook Form, or the state from `useActionState`. Show general failures in a summary, and move focus to the first invalid field.

**Q5. How do you prevent double submission?** ⭐
Disable the button while submitting, using `isSubmitting` or `useFormStatus`. That only helps the UI, so also send an idempotency key that lets the server ignore repeats.

**Q6. How do you build a large dynamic form efficiently?**
Use `useFieldArray` with stable IDs as keys, isolate re-renders with `useWatch` or `Controller`, split long forms into steps, and virtualise very long lists of fields. Keep the configuration as data if the form is generated from a schema.

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

**Scenario:** *"A failed save shows nothing, and the error boundary never triggers."* Boundaries don't catch errors in event handlers. Add `try/catch` in the handler and show a toast, or call `showBoundary(error)` if the whole section can't continue.

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

### 13.4 Interview Questions

**Q1. What do error boundaries catch, and what don't they catch?** ⭐
They catch errors during rendering, in lifecycle methods and in constructors of their children. They don't catch errors in event handlers, async code, server rendering, or the boundary itself. Handle those with `try/catch` or by setting error state.

**Q2. How do you handle an error from an async call?** ⭐
Catch it and store it in state, or let React Query expose it as `error`. To send it to the nearest boundary, rethrow it during render, or use `showBoundary` from `react-error-boundary`.

**Q3. Where should you place error boundaries?** ⭐
At several levels: one at the top as a last resort, one per route, and one around each independent widget. A failed chart then doesn't take down the whole page.

**Q4. How do you report frontend errors in production?** ⭐
Use a tool like Sentry, with `componentDidCatch` or `onError` on boundaries, plus global `error` and `unhandledrejection` listeners. Upload source maps privately so stack traces are readable, tag errors with the release, and scrub personal data.

**Q5. What should the user see when something fails?**
A clear message, a way to recover such as a retry button, and as much of the working UI as possible. Resetting a boundary needs a `resetKeys` value or a remount, so a retry actually re-runs the failed part.

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

**What to use at each level**

| Layer | Tests | Tools | How many |
|---|---|---|---|
| Static | Types, lint rules | TypeScript, ESLint | Everywhere |
| Unit | Pure functions, hooks with logic | Vitest / Jest | Many |
| Integration | A feature with real children and a mocked network | RTL + MSW | **Most of your effort** |
| E2E | Critical user journeys | Playwright / Cypress | Few |
| Visual | Appearance changes | Chromatic, Percy, Playwright screenshots | Key components |

**What's worth testing first:** the flows that would cost the most if they broke (login, checkout, payments), bugs you've already fixed, and complex logic. A coverage percentage on its own says little: 100% coverage can still miss the behaviour that matters.

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

**Scenario:** *"A test passes, but the button doesn't work for real users."* It may be calling the handler directly. Render the component, and use `userEvent` to click the button found with `getByRole`, then assert on what the user sees.

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

**Scenario:** *"How do you test the error state of a data-fetching component?"* Override the handler for that test with `server.use(...)` to return a 500, then assert that the error message appears. Handlers reset after each test.

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

**Scenario:** *"A component using Context crashes in tests."* It needs its Provider. Create a shared `renderWithProviders` helper that wraps components in the router, query client and theme, with a new query client for each test.

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

### 14.7 Interview Questions

**Q1. What do you test in a React app?** ⭐
User-visible behaviour: what renders, what happens on interaction, and the loading, error and empty states. Avoid testing internal state or implementation details. Put most effort into integration tests, and keep E2E tests for critical journeys such as login and checkout.

**Q2. `getBy` vs `queryBy` vs `findBy`?** ⭐
`getBy` throws if the element isn't found, so use it when the element should exist. `queryBy` returns `null`, so use it to assert something is absent. `findBy` returns a promise and retries, so use it for elements that appear asynchronously.

**Q3. Why use MSW instead of mocking `fetch` or axios?** ⭐
MSW intercepts at the network level, so your real data-fetching code runs. Tests then keep working if you change from axios to `fetch`, and the same handlers can be used in development and Storybook.

**Q4. What does the `act()` warning mean?** ⭐
State changed outside a tracked update, often because a test ended before an async update finished. Wait for the result with `findBy` or `waitFor`. RTL's helpers already wrap updates in `act`, so you rarely call it directly.

**Q5. How do you deal with flaky E2E tests?** ⭐
Wait for conditions, never fixed sleeps. Use stable selectors such as roles or test IDs, isolate test data so tests don't share state, and mock unstable third parties. Use traces from failed runs to fix the cause, rather than relying on retries.

**Q6. Are snapshot tests useful?**
Only for small, stable output. Large snapshots get approved without being read, so they catch little and fail on harmless changes. Prefer explicit assertions, and use visual regression tools for appearance.

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

**How the options compare**

| | Runtime CSS-in-JS (styled-components, Emotion) | Build-time CSS-in-JS (vanilla-extract, Panda CSS, StyleX) | CSS Modules | Tailwind |
|---|---|---|---|---|
| Styles generated | In the browser, at render | At build time | At build time | At build time |
| JS runtime cost | Yes | None or minimal | None | None |
| Dynamic styles | Very easy, using props | CSS variables or variants | CSS variables or classes | Conditional classes |
| Server Components | Needs client components and setup | Works well | Works well | Works well |
| Type safety | Partial | Strong | Limited | Through tooling |

**Dynamic values without a runtime:** set a CSS variable, so a change doesn't generate a new class.

```jsx
// ❌ Runtime: a new class can be generated for every value
const Bar = styled.div`width: ${p => p.percent}%;`;

// ✅ Static CSS, with a dynamic variable
<div className={styles.bar} style={{ '--percent': `${percent}%` }} />
```

```css
.bar { width: var(--percent); }
```

**Scenario:** *"Large lists are slow, and profiling shows time spent in styled-components."* Dynamic styles are being generated for every row. As a first step, swap prop-based styles for CSS variables or `data-` attributes. Longer term, migrate one component at a time to CSS Modules, Tailwind or a build-time library.

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

### 15.4 Interview Questions

**Q1. How do you choose a styling approach?** ⭐
Consider team skills, SSR needs, performance and the design system. CSS Modules give scoped plain CSS, Tailwind gives speed and consistency through utilities, and build-time CSS-in-JS keeps typed, component-level styles with no runtime cost.

**Q2. What are the downsides of runtime CSS-in-JS?** ⭐
It adds JS to the bundle, generates and injects styles during render, needs extra SSR setup, and doesn't fit well with Server Components and concurrent rendering. Build-time tools avoid this.

**Q3. What are the pros and cons of Tailwind?** ⭐
Pros: fast development, consistent design tokens, no naming, and small CSS because unused classes are dropped. Cons: long class lists, a learning curve, and repeated patterns unless you extract components.

**Q4. How do CSS Modules scope styles?** ⭐
The build tool rewrites each class name to a unique one, such as `.button_x3f9`, so classes can't collide between components. You still write normal CSS.

**Q5. How do you implement dark mode?** ⭐
Define colours as CSS variables and switch them with a `data-theme` attribute, defaulting to `prefers-color-scheme`. Save the choice, and set it with a small inline script before first paint, to avoid a flash of the wrong theme.

**Q6. How would you build a design system?**
Start with design tokens, then accessible primitives, then composed components. Document them in Storybook, publish as a versioned package, and use visual regression tests. Headless libraries like Radix or React Aria save a lot of accessibility work.

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

**Scenario:** *"A `Card` should accept text, icons or a list as its content."* Type it as `children: React.ReactNode`. `JSX.Element` would be too narrow, because it rejects strings and `null`.

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

**Scenario:** *"`inputRef.current.focus()` shows 'possibly null'."* Refs start as `null` until mounted. Type it as `useRef<HTMLInputElement>(null)` and use `inputRef.current?.focus()`.

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

**Scenario:** *"Our `<Button>` wrapper keeps needing more props, such as `aria-label` and `type`."* Extend `ComponentProps<'button'>` and spread `...rest` onto the element. All native props then work with no extra code.

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

**Scenario:** *"A reusable `Select` returns `any`, so consumers lose their types."* Make it generic: `Select<T>` with `options: T[]` and `onChange: (value: T) => void`. TypeScript infers `T` from the options.

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

**Scenario:** *"A component has `isLoading`, `isError` and `data`, and bugs show a spinner and an error together."* Replace the booleans with one union on `status`. Impossible states can no longer be represented, and `switch` handles every case.

---

### 16.6 Interview Questions

**Q1. `interface` vs `type` for props?** ⭐
Both work for props. `interface` can be extended and merged, while `type` also handles unions, tuples and mapped types. Most teams pick one convention, and use `type` wherever unions are needed.

**Q2. How do you type `children` and event handlers?** ⭐
Use `React.ReactNode` for `children`, because it accepts anything renderable. For events, use types such as `React.ChangeEvent<HTMLInputElement>` and `React.FormEvent<HTMLFormElement>`. Inline handlers usually infer these for you.

**Q3. Should you use `React.FC`?** ⭐
It's no longer harmful, because since React 18's types it doesn't add `children` automatically. Many teams still prefer typing props directly on the function, since it's simpler and works better with generics.

**Q4. What is a discriminated union, and why use it for state?** ⭐
A union whose members share a literal field, such as `status: 'loading' | 'success' | 'error'`. TypeScript narrows the type per case, so `data` only exists on success. Impossible combinations like loading plus error can't be written.

**Q5. How do you write a generic component?** ⭐
Add a type parameter: `function List<T>({ items, renderItem }: Props<T>)`. TypeScript then infers `T` from `items`, so `renderItem` is correctly typed. It's ideal for tables, selects and lists.

**Q6. How do you make a component accept all native button props?**
Extend `React.ComponentProps<'button'>` (or `ComponentPropsWithoutRef`), then spread the remaining props onto the element. Consumers get `disabled`, `onClick` and every other native attribute with no extra code.

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

**Scenario:** *"Users can add a website link to their profile."* Someone could enter `javascript:alert(1)`. Parse it with `new URL()` and allow only `http:` and `https:`, on both the client and the server.

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

**Scenario:** *"A new CSP breaks analytics and inline scripts in production."* Roll it out with `Content-Security-Policy-Report-Only` first, review the reports, add the required sources or nonces, then enforce it.

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

**Scenario:** *"A Stripe secret key is found in the production bundle."* It was exposed through a `NEXT_PUBLIC_` or `VITE_` variable. Rotate the key immediately, move the call behind your server, and add secret scanning to CI.

---

### 17.6 React Server Components Vulnerability (CVE-2025-55182)

**Explanation**
In December 2025, a critical flaw nicknamed **React2Shell** was disclosed in React Server Components. It allowed **unauthenticated remote code execution**: an attacker could send a crafted request to a server endpoint and run code on the server. It was rated the maximum severity (CVSS 10.0).

- **Cause:** unsafe deserialisation in the RSC "Flight" protocol, which handles data sent to the server.
- **Affected:** `react-server-dom-webpack`, `-parcel` and `-turbopack` in React 19.0, 19.1.0, 19.1.1 and 19.2.0, plus frameworks that bundle them, such as Next.js App Router (tracked there as CVE-2025-66478).
- **First fixes:** React 19.0.1, 19.1.2 and 19.2.1, with matching Next.js patches. Related issues were found afterwards, so install the **latest patched release**, not just the first fix.
- **Not affected:** apps with no server-side React at all, such as a pure client-side SPA.

**Why it matters in interviews:** RSC moves React onto the server, so the frontend now has **server-side attack surface**. Server Actions and Server Functions are public HTTP endpoints, so treat them like any other API.

```tsx
'use server';

export async function deleteOrder(orderId: string) {
  const user = await requireUser();                // authenticate
  const order = await db.orders.find(orderId);
  if (order.userId !== user.id) throw new Error('Forbidden');   // authorise
  await db.orders.delete(orderId);
}
```

**Scenario:** *"A critical CVE drops for a framework you run in production. What do you do?"*
1. **Check exposure**: which apps and workspaces use the affected packages, including through lockfiles and vendored products?
2. **Patch and redeploy**: updating `package.json` isn't enough; you have to rebuild and ship.
3. **Add temporary protection**: WAF rules help, but they don't replace patching.
4. **Assume compromise if exposed**: review logs and rotate secrets.
5. **Improve the process**: automated dependency alerts, and a way to ship an urgent patch quickly.

---

### 17.7 Interview Questions

**Q1. How does React protect against XSS, and where can it still happen?** ⭐
React escapes values rendered in JSX, so strings can't inject HTML. Risks remain with `dangerouslySetInnerHTML`, user-supplied `javascript:` URLs, unsafe third-party scripts, and direct DOM writes. Sanitise HTML with DOMPurify and validate URL schemes.

**Q2. Where should you store auth tokens?** ⭐
Prefer `HttpOnly`, `Secure`, `SameSite` cookies, because JavaScript can't read them, so XSS can't steal them. `localStorage` is readable by any script. Cookies then need CSRF protection. Short-lived access tokens held in memory, with a refresh token in an `HttpOnly` cookie, is another common approach.

**Q3. What is CSRF, and how do you prevent it?** ⭐
A malicious site makes the browser send a request to your site with the user's cookies. Prevent it with `SameSite` cookies, CSRF tokens and `Origin` checks. APIs that use tokens in the `Authorization` header aren't exposed in the same way.

**Q4. What is a Content Security Policy?** ⭐
A response header that lists which sources of scripts, styles and other resources the browser may load. It limits the damage from XSS. Avoid `unsafe-inline`, use nonces or hashes, and start in report-only mode to find problems first.

**Q5. Can you keep API keys in a React app?** ⭐
No. Everything in the bundle is public, including `VITE_*` and `NEXT_PUBLIC_*` variables. Put secrets on the server and call third parties through your own backend.

**Q6. Is hiding admin buttons enough for security?**
No. Client-side checks only improve the UX. The server must check permissions on every request, because anyone can call the API directly.

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

**Scenario:** *"Clickable `<div>`s can't be used with a keyboard."* Replace them with `<button>` for actions and `<a href>` for navigation. You get focus and keyboard support with no extra code, and you can restyle them with CSS.

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

**Scenario:** *"An icon-only delete button is announced as just 'button'."* Add `aria-label="Delete order"`, and mark the icon `aria-hidden="true"`. Include what's being deleted, because screen reader users often navigate by a list of buttons.

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

**Scenario:** *"After a route change, keyboard users are stuck at the old focus position."* SPAs don't reset focus. On navigation, move focus to the new page's `<h1>` (with `tabIndex={-1}`), and announce the new title.

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

**Scenario:** *"axe reports no issues, but a screen reader user can't complete checkout."* Automated tools can't judge the order of focus or how understandable the flow is. Test the journey by keyboard and with VoiceOver or NVDA, and add those checks to your definition of done.

---

### 18.6 Interview Questions

**Q1. Why prefer semantic HTML over ARIA?** ⭐
Native elements include keyboard support, focus and screen reader behaviour for free. A `<button>` is focusable and activates with Enter and Space, while a `<div role="button">` needs all of that added by hand. The first rule of ARIA is not to use it if a native element will do.

**Q2. How do you make a modal accessible?** ⭐
Use `role="dialog"` and `aria-modal`, label it, move focus inside when it opens, trap Tab within it, close on Escape, and return focus to the trigger when it closes. The native `<dialog>` element with `showModal()` handles much of this.

**Q3. How do you make a custom component keyboard accessible?** ⭐
Follow the ARIA Authoring Practices pattern for that widget. Make it focusable, handle the expected keys (such as arrows in tabs and menus), keep focus visible, and manage roving `tabindex` for groups.

**Q4. How do screen readers learn about dynamic updates?** ⭐
Through live regions: `aria-live="polite"` or `role="status"` for updates that can wait, and `role="alert"` for urgent ones. The region must already be in the DOM before its content changes.

**Q5. How do you test accessibility?** ⭐
Automated tools (axe, eslint-plugin-jsx-a11y, Lighthouse) find only a portion of issues. Add keyboard-only testing and real screen reader checks with VoiceOver or NVDA. RTL's `getByRole` queries also encourage accessible markup.

**Q6. What should an image's `alt` text be?**
A description of what the image conveys. Use `alt=""` for decorative images, so screen readers skip them, and don't begin with "image of".

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

For full worked examples, see [Section 29](#29-frontend-system-design-case-studies).

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

**What happens in a production build**

```
Source (TS/JSX) ─► transpile ─► resolve imports ─► tree shake ─► split into chunks ─► minify ─► hashed files + source maps
```

**Choosing a tool**

| Situation | Choice |
|---|---|
| New SPA | Vite |
| Next.js app | Turbopack (the default in Next 16) |
| Large Webpack app wanting speed without a rewrite | Rspack |
| Publishing a component library | Vite library mode, tsup or Rollup |
| Heavy Module Federation setup | Webpack or Rspack |

```js
// vite.config.js
export default defineConfig({
  plugins: [react()],
  build: {
    sourcemap: 'hidden',             // upload to Sentry without exposing them publicly
    rollupOptions: {
      output: {
        manualChunks: { charts: ['recharts'] },   // keep a heavy library in its own chunk
      },
    },
  },
});
```

**Tree shaking needs ES modules.** It works with `import`/`export` because they can be analysed statically, not with CommonJS `require`. Libraries should also mark `"sideEffects": false` in `package.json` when it's safe. Import only what you need (`lodash-es` rather than all of `lodash`), and check the results with a bundle analyser.

**Transpiling isn't type checking:** esbuild and SWC strip types without checking them, so run `tsc --noEmit` in CI.

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

**A safe migration, step by step**

| Step | What you do |
|---|---|
| 1. Baseline | Record current metrics: bugs, Web Vitals, build time, developer pain points |
| 2. Safety net | Add E2E tests around critical flows before changing code |
| 3. Stop the growth | New code uses the new approach, enforced by a lint rule |
| 4. Run both | Old and new side by side, with a clear boundary between them |
| 5. Move in slices | One route or feature at a time, behind a feature flag |
| 6. Measure and roll back | Compare metrics, and keep rollback simple |
| 7. Finish | Delete the old code and dependencies, so the migration doesn't stay half done |

Step 7 is the one teams usually skip, and two systems permanently in use is worse than either alone. Track remaining work with a visible number, such as files still using the old pattern.

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

### 19.10 Interview Questions

**Q1. How do you structure a large React codebase?** ⭐
Organise by feature rather than file type: each feature keeps its components, hooks, API calls and tests together, with a clear public entry point. Shared UI and utilities live separately. Enforce boundaries with lint rules so features don't reach into each other.

**Q2. How do you approach a frontend system design question?** ⭐
Use RADIO: requirements, architecture, data model, interface, optimisations. Clarify the scope first, then spend most of your time on the hard parts of that product, covering performance, accessibility, failure handling and trade-offs.

**Q3. What are micro-frontends, and when should you use them?** ⭐
Independently built and deployed frontend parts, composed into one app, for example with Module Federation. They help many teams release independently. They cost more in complexity, bundle size and UI consistency, so they're a poor default for a small team.

**Q4. Monorepo or multiple repos?** ⭐
A monorepo makes shared code, atomic changes and consistent tooling easier, using Nx or Turborepo to build only what changed. Multiple repos give stronger isolation but make shared changes harder to coordinate.

**Q5. Why is Vite faster than Webpack in development?** ⭐
Webpack bundles the app before serving it. Vite serves native ES modules on demand and pre-bundles dependencies with a fast native tool, so startup barely grows with app size and hot updates stay quick.

**Q6. How do you release frontend features safely?**
Use feature flags, gradual rollouts, preview deployments and automated tests in CI. Monitor errors and Web Vitals by release, and keep rollback simple by retaining the previous build.

**Q7. How do you add internationalisation?**
Move strings into translation files and use a library such as react-i18next or FormatJS. Use `Intl` for dates, numbers and plurals, leave room for longer text, and support right-to-left layouts.

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

**Scenario:** *"Build a `once()` function."* A closure stores whether it has run and its result: `const once = fn => { let done, result; return (...a) => done ? result : (done = true, result = fn(...a)); };`.

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

**Scenario:** *"`this` is undefined inside a method passed as a callback."* Passing `obj.method` detaches it from `obj`. Use an arrow function, `obj.method.bind(obj)`, or define the method as an arrow class field.

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

**Scenario:** *"Five independent API calls take 5 seconds with `await` in a loop."* They run one after another. Start them together with `Promise.all`, or `Promise.allSettled` if one failure shouldn't cancel the rest. For hundreds of calls, limit how many run at once.

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

**Scenario:** *"A property added to `Array.prototype` shows up in every `for...in` loop."* Inherited enumerable properties are included. Don't modify built-in prototypes, and use `for...of` for arrays.

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

```js
// Temporal dead zone: let/const exist from the start of the block, but can't be used yet
console.log(a);          // undefined (var is initialised on hoisting)
console.log(b);          // ReferenceError
var a = 1;
let b = 2;

// Block scope
if (true) { var x = 1; let y = 2; }
console.log(x);          // 1
console.log(y);          // ReferenceError

// A loop creates a new let binding on each iteration
for (var i = 0; i < 3; i++) setTimeout(() => console.log(i));   // 3 3 3
for (let j = 0; j < 3; j++) setTimeout(() => console.log(j));   // 0 1 2

// const prevents reassignment, not mutation
const user = { name: 'Asha' };
user.name = 'Ravi';      // allowed
user = {};               // TypeError
```

**Scenario:** *"Referencing a `const` throws 'Cannot access before initialization'."* It's in the temporal dead zone: the name is known to the block, but it hasn't been initialised. Move the declaration above its first use.

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

**Scenario:** *"Editing a copied form object also changes the original."* A spread copy is shallow, so nested objects are shared. Copy each level you change, use `structuredClone` for a full copy, or use Immer.

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

**Scenario:** *"A debounced search in a component never fires, or fires on every key."* A new debounced function is being created on each render. Create it once with `useMemo` or a ref, and cancel it on unmount.

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

**Scenario:** *"Implement `Promise.all` and handle an empty array."* It must resolve immediately with `[]`, keep results in the input order regardless of finish order, and accept non-promise values through `Promise.resolve`.

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

**Scenario:** *"`count && <List />` renders a stray 0 on the page."* `&&` returns the left value, and React renders the number `0`. Use `count > 0 && <List />`. Similarly, `value || 10` replaces a valid `0`, so use `value ?? 10`.

---

### 20.11 Interview Questions

**Q1. What is a closure?** ⭐
A function that remembers variables from the scope where it was created, even after that scope has finished. Closures power private state, callbacks and Hooks, and they're also the cause of stale closures in React.

**Q2. Explain the event loop.** ⭐
Synchronous code runs on the call stack. When it's empty, **all** microtasks run (promise callbacks, `queueMicrotask`), then the browser may render, then one macrotask runs (such as `setTimeout`). So a resolved promise always runs before a `setTimeout(fn, 0)`.

**Q3. How does `this` work?** ⭐
It depends on how a function is called: the object before the dot, the explicit value with `call`, `apply` or `bind`, or the new object with `new`. Arrow functions don't have their own `this`, and use the one from the surrounding scope.

**Q4. `Promise.all` vs `allSettled` vs `race` vs `any`?** ⭐
`all` waits for everything and rejects on the first failure. `allSettled` waits for everything and reports each outcome. `race` settles with the first one to finish, either way. `any` resolves with the first success and rejects only if all fail.

**Q5. Debounce vs throttle?** ⭐
Debounce runs after activity stops, which suits search input. Throttle runs at most once per interval, which suits scroll and resize handlers.

**Q6. What's the output of a `var` loop with `setTimeout`?** ⭐
`for (var i = 0; i < 3; i++) setTimeout(() => console.log(i))` prints `3 3 3`, because all callbacks share one `i`. With `let`, each iteration gets its own binding, so it prints `0 1 2`.

**Q7. How does prototypal inheritance work?**
Each object has a link to a prototype. When a property isn't found, JavaScript looks up that chain. Classes are mostly syntax over this mechanism.

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

**A time plan for a 60-minute round**

| Time | Do this |
|---|---|
| 0–5 min | Clarify requirements, and agree what's in scope |
| 5–10 min | Plan components and state out loud |
| 10–35 min | Build the core feature so it works end to end |
| 35–50 min | Loading, error and empty states, edge cases, keyboard support |
| 50–60 min | Test by hand, clean up, and explain what you'd do next |

**Mistakes that cost the most**
- Starting to code before understanding the problem.
- Spending 15 minutes on CSS before anything works.
- Storing derived values in state, or copying props into state.
- Going silent when stuck. Say what you're trying instead.
- Forgetting to clean up timers and listeners.
- Using a key based on the array index in a list that changes.

**Scenario:** *"The requirement is vague: 'build a file explorer'."* Don't start typing straight away. Ask whether the data is local or from an API, whether folders load on demand, and whether rename or delete is required. Then state your assumptions and begin with the smallest useful version.

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

**Scenario:** *"The interviewer asks you to support 100,000 results."* Limit results on the server, add caching per query, and virtualise the list if it must stay long.

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

**Scenario:** *"Now sync the todos across browser tabs."* Listen for the `storage` event, or use `BroadcastChannel`, and dispatch a `SYNC` action. Decide what happens when two tabs edit at once.

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

**Scenario:** *"Users report duplicate items while scrolling."* Fast scrolling triggers several requests for the same page, or offset pagination shifts. Block requests while one is loading, use cursor pagination, and de-duplicate by ID.

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

**Scenario:** *"Add support for a modal opening another modal."* Keep a stack of open modals. Escape closes only the top one, focus returns to the previous one, and body scroll stays locked until the stack is empty.

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

**Scenario:** *"Let the parent control which panel is open."* Accept optional `openItems` and `onChange` props, and fall back to internal state when they aren't provided, which is the controlled and uncontrolled pattern from 5.5.

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

**Scenario:** *"Add half-star ratings and make it accessible."* Use the pointer position within a star to choose `.5`, and expose it as a radio group, or `role="slider"` with arrow-key support.

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

**Scenario:** *"The countdown is wrong after the tab has been in the background."* Browsers slow down timers in hidden tabs. Store an end timestamp and calculate the remaining time from `Date.now()` on each tick.

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

**Scenario:** *"Replying to a deeply nested comment is slow, and the update code is complicated."* Store comments in a flat map with `parentId` and `childIds`. An update then changes one entry, and only the affected branch re-renders.

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

**Scenario:** *"Now the table has 1 million rows."* Move sorting, filtering and pagination to the server, keep those settings in the URL, and show old data while the next page loads.

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

**Scenario:** *"Extend it to an N×N board with undo."* Generate the winning checks from `n`, and store a history array of moves. Undo moves an index back, and the current board is derived from it.

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

**Scenario:** *"Add a pedestrian button that shortens the green light."* Put the transitions in a reducer, with a `PEDESTRIAN_REQUEST` event that changes the next timeout. The timer Effect re-runs on state change, and cleanup prevents overlapping timers.

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

**Scenario:** *"Autoplay should stop when the user hovers or tabs in."* Pause on hover and focus, and when the tab is hidden. Provide a pause button and respect `prefers-reduced-motion`, as accessibility guidelines require.

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

**Scenario:** *"You get a problem you've never seen."* Fall back on the same process: define the state, derive what you can, list the events that change it, then build the UI. Most problems combine patterns you already know.

---

### 21.14 Interview Questions

**Q1. How do you approach a machine coding round?** ⭐
Clarify requirements, describe the component tree and state, then build the simplest working version first. After that, add loading, error and empty states, edge cases and accessibility. Talk through your decisions as you go.

**Q2. What are interviewers looking for?** ⭐
Working code, sensible components, minimal state in the right place, correct keys, cleanup of timers and listeners, and clear communication. A simple finished solution beats an ambitious unfinished one.

**Q3. How do you avoid bugs with timers in React?** ⭐
Clear them in the Effect's cleanup, use functional updates so they don't depend on stale state, and keep the timer ID in a ref. For accurate countdowns, calculate from timestamps, because intervals drift.

**Q4. How do you decide what state a component needs?** ⭐
Find the minimum set of values, and calculate everything else during render. Filtered lists and totals, for example, aren't state. Keep state as close as possible to where it's used, and move to `useReducer` when updates are related.

**Q5. What if you're running out of time?** ⭐
Get the core working and say what's left. Explain how you'd add it, for example: "Next, I'd virtualise this list and add tests." Interviewers value prioritisation, so don't start something you can't finish.

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

**Stories worth preparing**

| Story | Questions it answers |
|---|---|
| Hardest technical problem | Hard bug, biggest challenge, proudest work |
| Performance win | Optimisation, measuring impact |
| Disagreement | Conflict, influence, pushback |
| Mistake or failure | Failure, learning, accountability |
| Leadership or mentoring | Ownership, helping others |
| Tight deadline | Prioritisation, trade-offs |

**A short example**
> **S:** Our checkout had a 12% drop-off on mobile.
> **T:** I was asked to find out why and fix it within a sprint.
> **A:** I checked session recordings and field data, and found that a 900 KB bundle delayed the payment form. I split the route, lazy-loaded the address autocomplete and removed a date library.
> **R:** Time to interactive fell by about half, and the drop-off fell to 8%.

Numbers in your own stories must be true, and you should be able to explain how you measured them.

**Scenario:** *"I've mostly worked in a team. How do I talk about my own contribution?"* Give the team context in one sentence, then say "I" for what you personally decided, built or led. If you're asked for a detail, be honest about who owned it.

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

**Scenario:** *"My project is under an NDA."* Describe the architecture and scale in general terms, such as "a B2B dashboard with around 50k users". Leave out client names, internal details and exact business numbers.

---

### 22.3 "Tell Me About a Performance Problem You Solved"

**Example (adapt to your experience):**
> **S:** Our order management table (2,000 rows, inline editing) took ~800ms to respond to each keystroke; INP was in the "poor" range.
> **T:** I owned getting it under 200ms without removing features.
> **A:** I profiled with React DevTools and found every row re-rendering on each keystroke: the edit state lived at the table level and each row got inline callbacks. I moved the edit state into the row, memoised the rows with stable callbacks (`useCallback` + ID-based handlers), and virtualized the table with TanStack Virtual so only ~30 rows were rendered. I also moved the totals calculation into `useMemo` keyed on the data version.
> **R:** Interaction time dropped from 800ms to ~40ms, INP p75 went from 650ms to 120ms, and support tickets about "laggy table" stopped. I wrote a short internal guide on profiling that two other teams used.

**Scenario:** *"I don't have exact performance numbers."* Say so and give honest estimates, like "from around 3 seconds to under 1 in our testing". Then explain how you'd measure it now. Don't invent precise figures, because follow-up questions will expose them.

---

### 22.4 "Tell Me About a Hard Bug You Debugged"

**Example:**
> **S:** Users occasionally saw another customer's details for a moment when switching quickly between customers in the CRM.
> **T:** It was a P1 (privacy risk); I took ownership.
> **A:** I reproduced it by throttling the network: request A (slow) resolved after request B and overwrote the state, a **race condition** in a `useEffect` fetch. I fixed it straight away with an `AbortController` cleanup, wrote a regression test using MSW with delayed responses, then led the migration of that module to React Query (keyed caching removes this whole class of bug). I also added an ESLint rule warning about fetch-in-effect without cleanup.
> **R:** Zero reports of it since; the module's data code shrank by ~40%; the pattern became our team standard.

**Scenario:** *"They ask how you'd stop this class of bug happening again."* Move beyond the one fix: a regression test, a lint rule, a shared helper, or a change in the review checklist.

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

**Scenario:** *"Your decision turned out to be wrong."* Say so. Explain what you knew then, what changed, how you corrected it, and how you decide differently now. It's usually a stronger answer than one with no mistakes.

---

### 22.6 Code Review, Mentoring & Disagreements

**Code review: what do you look for?**
Correctness and edge cases (loading, error, empty) → accessibility → performance problems (unneeded effects, missing keys, re-render causes) → security (`dangerouslySetInnerHTML`, secrets) → test coverage of behaviour → readability and naming → consistency with team patterns. Be kind and specific, separate "blocking" from "nit", explain the *why*, and suggest code.

**Mentoring example:**
> "A junior dev's PRs kept getting large review comment threads about effects. Instead of fixing them in review, I paired with them for an hour on 'You might not need an effect', and we refactored one of their components together. I then gave them ownership of a small feature with a check-in every two days. In two months their PRs needed half the revisions, and they presented the pattern at our frontend guild meeting."

**Disagreement example:**
> "A senior backend engineer wanted the frontend to poll every 2s for job status. I was concerned about server load and battery. I didn't argue opinions; I collected data: 5k concurrent users × 0.5 req/s = 2.5k req/s just for polling. I proposed SSE, with polling as a fallback. We agreed to an experiment behind a flag, and SSE cut the requests by 95%. I made sure we presented it as a joint decision."

**Scenario:** *"A senior engineer rejects your PR with no explanation."* Ask for their reasoning privately, and describe the trade-off you were making. If you still disagree, suggest a small experiment or bring in a tech lead. Keep it about the code.

---

### 22.7 Questions to Ask the Interviewer

- What does the frontend architecture look like? React version, SSR or not?
- How is frontend quality measured: Web Vitals, error budgets, test coverage?
- How do frontend and backend/design teams work together? Is there a design system?
- What's the biggest frontend technical challenge right now?
- How are decisions about new libraries or patterns made?
- What does success look like for this role in the first 6 months?
- How is on-call / production support handled for the frontend?

**Scenario:** *"They ask whether you have questions, and everything's been covered."* Still ask one or two, such as what would make someone successful here in six months. Saying "no questions" can come across as low interest.

---

### 22.8 Interview Questions

**Q1. Walk me through your project's frontend architecture.** ⭐
Start with the product and scale. Then cover rendering, state, data fetching, testing and deployment. Finish with one decision you're proud of and one you'd change. Keep it to 2–3 minutes and let them choose where to go deeper.

**Q2. Tell me about a difficult bug you solved.** ⭐
Pick a bug with real impact. Explain how you reproduced it, how you narrowed it down, what the cause was, and how you stopped it recurring, such as a regression test or a lint rule.

**Q3. Describe a performance improvement you made.** ⭐
State the problem with a number, how you measured it, what you changed and why, then the measured result. Without before and after numbers, the story is much weaker.

**Q4. Tell me about a time you disagreed with a colleague.** ⭐
Show that you listened and used evidence. Explain the other person's view fairly, how you tested the options, and how you reached a decision. Finish by showing the working relationship stayed healthy.

**Q5. Tell me about a mistake or failure.** ⭐
Pick a real mistake that's now behind you. Take responsibility without blaming anyone, explain how you fixed it, and describe what you changed so it wouldn't happen again.

**Q6. How do you mentor junior developers?**
Give a specific example: pairing rather than simply correcting, explaining the why in reviews, and giving growing ownership. Finish with how that person improved.

**Q7. Why do you want to leave your current role?**
Stay positive and focus on what you're moving towards, such as scale or ownership. Don't criticise your employer, and connect your answer to this role.

---

## 23. HTML & CSS Fundamentals

### 23.1 HTML Essentials: Document Structure, Scripts & Metadata

**Explanation**
For semantic elements, see [18.1](#181-semantic-html-first). These are the other HTML points interviewers ask about.

```html
<!doctype html>                              <!-- standards mode, not quirks mode -->
<html lang="en">                             <!-- language for screen readers and translation -->
<head>
  <meta charset="utf-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1" />  <!-- needed for responsive layouts -->
  <link rel="preconnect" href="https://api.example.com" />
  <link rel="stylesheet" href="/app.css" />
  <script src="/app.js" defer></script>
</head>
```

**`<script>` loading**

| Attribute | Download | Runs | Order kept | Use for |
|---|---|---|---|---|
| none | Blocks HTML parsing | Immediately | Yes | Avoid in `<head>` |
| `async` | In parallel | As soon as it's ready | **No** | Independent scripts (analytics) |
| `defer` | In parallel | After parsing, before `DOMContentLoaded` | **Yes** | App code |
| `type="module"` | In parallel | Deferred by default | Yes | ES modules |

**Resource hints**

| Hint | Purpose |
|---|---|
| `preconnect` | Open the connection to an origin early |
| `preload` | Fetch a resource needed for **this page** at high priority |
| `prefetch` | Fetch a resource likely needed for the **next page** at low priority |

**Scenario:** *"The hero image is slow to appear, and it's set as a CSS background."*
The browser only discovers it after downloading and parsing the CSS. Use an `<img>` in the HTML with `fetchpriority="high"`, or add `<link rel="preload" as="image">`. Don't lazy-load it, because it's your LCP element.

---

### 23.2 The Box Model

**Explanation**
Every element is a box made of **content → padding → border → margin**.

- **`box-sizing: content-box`** (the default): `width` covers content only, so padding and border are added on top.
- **`box-sizing: border-box`**: `width` includes padding and border, which is far easier to work with.

```css
*, *::before, *::after { box-sizing: border-box; }

.card { width: 300px; padding: 20px; border: 1px solid; }
/* content-box: total width 342px | border-box: total width 300px */
```

**Margin collapsing:** vertical margins between adjacent block elements merge into the larger of the two (20px + 30px gives 30px, not 50px). It also happens between a parent and its first or last child. It doesn't happen in flex or grid containers, or inside a new **block formatting context** (`display: flow-root`).

**Scenario:** *"A child's `margin-top` is pushing the parent down instead of moving inside it."*
Parent and child margins are collapsing. Add `display: flow-root` to the parent, give it padding or a border, or make it a flex container.

---

### 23.3 The Cascade & Specificity

**Explanation**
When several rules target the same element, the browser decides in this order: **origin and `!important` → `@layer` → specificity → source order** (the last one wins).

**Specificity** is compared as three columns, left to right:

| Selector | IDs | Classes, attributes, pseudo-classes | Elements, pseudo-elements |
|---|---|---|---|
| `p` | 0 | 0 | 1 |
| `.btn` | 0 | 1 | 0 |
| `.nav a:hover` | 0 | 2 | 1 |
| `#main .btn` | 1 | 1 | 0 |

- Inline styles beat all selectors. `!important` beats inline styles.
- One ID beats any number of classes, because the columns don't carry over.
- `:where()` adds **zero** specificity. `:is()` and `:not()` take the specificity of their most specific argument.
- **`@layer`** orders groups of styles, so later layers win regardless of specificity.

```css
@layer reset, components, utilities;

@layer components { #app .button { color: blue; } }
@layer utilities  { .text-red { color: red; } }    /* wins, despite lower specificity */
```

**Inheritance:** text properties like `color` and `font` inherit. Layout properties like `margin` and `border` don't.

**Scenario:** *"Our override isn't working unless we add `!important`."*
The original selector has higher specificity. Check it in DevTools, then reduce the specificity of the original, or use `@layer` for third-party CSS. Using `!important` starts an arms race that gets worse over time.

---

### 23.4 Display, Positioning & Stacking Contexts

**Explanation**

| `position` | Reference point | In normal flow? |
|---|---|---|
| `static` | None; offsets are ignored | Yes |
| `relative` | Its own normal position | Yes (its space is kept) |
| `absolute` | Nearest **positioned** ancestor | No |
| `fixed` | The viewport | No |
| `sticky` | Acts `relative` until a scroll threshold, then sticks | Yes |

**Three `position` traps**
- `sticky` needs an offset such as `top: 0`, and stops working if an ancestor has `overflow: hidden` or `auto`.
- `fixed` is positioned relative to an ancestor with a `transform` or `filter`, not the viewport.
- `absolute` without a positioned ancestor goes up to the initial containing block.

**Stacking contexts:** `z-index` only compares elements **within the same stacking context**. A new context is created by a positioned element with a `z-index`, `opacity` below 1, `transform`, `filter`, `will-change`, or `isolation: isolate`.

```css
.card { transform: translateZ(0); }     /* creates a stacking context */
.card .tooltip { z-index: 9999; }       /* can't escape .card, however high the number */
```

**`display: none` vs `visibility: hidden` vs `opacity: 0`**

| | Takes up space | Clickable | Read by screen readers |
|---|---|---|---|
| `display: none` | No | No | No |
| `visibility: hidden` | Yes | No | No |
| `opacity: 0` | Yes | **Yes** | **Yes** |

**Scenario:** *"A modal with `z-index: 9999` still appears behind the header."*
The modal sits inside a stacking context whose parent ranks below the header. Raising `z-index` can't help. Render it at the document root with a Portal ([5.6](#56-portals)).

---

### 23.5 Flexbox

**Explanation**
Flexbox is a **one-dimensional** layout: a row or a column.

```css
.container {
  display: flex;
  flex-direction: row;             /* sets the main axis */
  justify-content: space-between;  /* along the main axis */
  align-items: center;             /* along the cross axis */
  flex-wrap: wrap;
  gap: 16px;
}
.item { flex: 1; }                 /* shorthand for flex: 1 1 0% */
```

| Property | Meaning |
|---|---|
| `flex-grow` | How much of the spare space to take |
| `flex-shrink` | How much to shrink when space runs out |
| `flex-basis` | Starting size before growing or shrinking |

**The classic bug:** flex items default to `min-width: auto`, so they refuse to shrink below their content.

```css
.sidebar { width: 240px; flex-shrink: 0; }
.content { flex: 1; min-width: 0; }   /* lets long text truncate instead of overflowing */
```

**Scenario:** *"Long text in a flex child overflows instead of truncating."*
Add `min-width: 0` to the flex item, and put `overflow: hidden; text-overflow: ellipsis; white-space: nowrap` on the text.

---

### 23.6 CSS Grid

**Explanation**
Grid is a **two-dimensional** layout: rows and columns at the same time.

```css
.layout {
  display: grid;
  grid-template-columns: 240px 1fr;
  grid-template-rows: auto 1fr auto;
  grid-template-areas:
    "header header"
    "sidebar main"
    "footer footer";
  min-height: 100dvh;
}
.header { grid-area: header; }

/* Responsive cards with no media queries */
.cards {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(240px, 1fr));
  gap: 16px;
}
```

- **`fr`** is a share of the remaining space.
- **`auto-fit`** collapses empty tracks so items stretch. **`auto-fill`** keeps empty tracks.
- **`subgrid`** lets nested elements line up with the parent's tracks.

**Flexbox or Grid?** Flexbox when **content** drives the layout (toolbars, tags, centring). Grid when the **layout** drives the content (page shells, card grids). They're often combined.

**Scenario:** *"Build a card grid that shows 4 columns on desktop and 1 on mobile."*
`repeat(auto-fit, minmax(240px, 1fr))` does it in one line. The browser fits as many 240px columns as it can.

---

### 23.7 Responsive Design & Modern CSS

**Explanation**
**Mobile-first:** write base styles for small screens, then add `min-width` media queries.

| Unit | Relative to | Use for |
|---|---|---|
| `rem` | Root font size | Font sizes and spacing (respects user settings) |
| `em` | Parent's font size | Spacing tied to the component's text |
| `%` | Parent's size | Fluid widths |
| `vw` / `vh` | Viewport | Full-screen sections |
| `dvh` / `svh` / `lvh` | Viewport, accounting for mobile browser bars | Use instead of `100vh` on mobile |

```css
/* Container queries: respond to the component's space, not the screen */
.card-wrapper { container-type: inline-size; }
@container (min-width: 400px) {
  .card { display: flex; }
}

/* Fluid type */
h1 { font-size: clamp(1.5rem, 4vw, 3rem); }

/* Theming with custom properties */
:root { --bg: #fff; --text: #111; }
@media (prefers-color-scheme: dark) {
  :root { --bg: #111; --text: #eee; }
}
body { background: var(--bg); color: var(--text); }

/* :has() selects a parent based on its children */
.field:has(input:invalid) { border-color: red; }
```

**Media vs container queries:** a media query looks at the viewport. A container query looks at the parent, which makes it the better choice for reusable components that appear in both a sidebar and a main area.

**Scenario:** *"A full-height mobile page is cut off behind the browser's address bar."*
`100vh` uses the largest viewport size, which ignores the browser bars. Use `100dvh`.

---

### 23.8 Interview Questions

**Q1. What's the difference between `content-box` and `border-box`?** ⭐
With `content-box`, `width` applies to content only, and padding and border add to the final size. With `border-box`, `width` includes them. Most projects set `border-box` globally because sizes are predictable.

**Q2. How is CSS specificity calculated?** ⭐
It's compared in three columns: IDs, then classes/attributes/pseudo-classes, then elements/pseudo-elements. The first column that differs decides. Inline styles beat selectors, `!important` beats inline, and equal specificity goes to the later rule. `@layer` order is checked before specificity.

**Q3. Flexbox vs Grid?** ⭐
Flexbox is one-dimensional and content-driven: good for navbars and aligning items. Grid is two-dimensional and layout-driven: good for page structure and card grids. Most apps use Grid for the outer layout and Flexbox inside components.

**Q4. How do you centre a div?** ⭐
`display: grid; place-items: center;` on the parent, or `display: flex; justify-content: center; align-items: center;`. For a block with a known width, `margin-inline: auto` centres it horizontally.

**Q5. Why might `z-index` not work?** ⭐
Either the element isn't positioned (and isn't a flex or grid item), or it's inside a stacking context ranked below the other element. `z-index` only competes within one context. Find what creates the context, or use a Portal.

**Q6. What's the difference between `async` and `defer`?** ⭐
Both download without blocking parsing. `async` runs as soon as it's downloaded, in any order. `defer` waits until parsing finishes and keeps the order. Use `defer` for app code and `async` for independent scripts.

**Q7. `em` vs `rem`?**
`rem` is relative to the root font size, so it's predictable. `em` is relative to the parent's font size, so it compounds when nested. Prefer `rem`, and use `em` for spacing that should scale with the component's own text.

**Q8. When would you use a container query rather than a media query?**
When a component's layout should depend on the space it's given, not the screen. The same card can then adapt whether it's in a narrow sidebar or a wide main column.

---

## 24. Browser Internals

### 24.1 The Critical Rendering Path

**Explanation**
These are the steps the browser takes to turn HTML, CSS and JS into pixels.

```
HTML ──► DOM ─────┐
                  ├──► Render Tree ──► Layout ──► Paint ──► Composite ──► pixels
CSS  ──► CSSOM ───┘
```

1. **Parse HTML** into the DOM.
2. **Parse CSS** into the CSSOM.
3. **Render tree**: combine them, keeping only visible nodes (`display: none` is left out).
4. **Layout**: calculate each element's size and position.
5. **Paint**: fill in the pixels for each layer.
6. **Composite**: combine the layers on the GPU.

**What blocks rendering**
- **CSS is render-blocking.** The browser won't paint until the CSSOM is ready.
- **Plain `<script>` tags are parser-blocking.** They pause HTML parsing, and they also wait for pending CSS because scripts can read styles.

**How to speed it up**
- Inline critical CSS and defer the rest.
- Use `defer` or `async` on scripts.
- Keep the initial HTML useful (SSR) so content doesn't wait for JS.
- `preconnect` to important origins, and `preload` the LCP image and key fonts.

**Scenario:** *"Our React SPA shows a blank white page for 3 seconds on mobile."*
With CSR, the HTML is an empty `<div>`, so nothing meaningful appears until JS downloads, parses and runs. Fixes: SSR or SSG for the first paint, code splitting to shrink the initial bundle ([7.4](#74-code-splitting-reactlazy-suspense-dynamic-import)), and preloading critical assets.

---

### 24.2 Reflow, Repaint & Compositing

**Explanation**
Different CSS changes cost different amounts of work:

| Change | Example properties | Pipeline work | Cost |
|---|---|---|---|
| **Reflow (layout)** | `width`, `height`, `top`, `margin`, `font-size` | Layout → Paint → Composite | Highest |
| **Repaint** | `color`, `background`, `box-shadow` | Paint → Composite | Medium |
| **Composite only** | `transform`, `opacity` | Composite | Lowest (GPU) |

That's why you animate with `transform` and `opacity`, not `left` or `width`.

**Layout thrashing** happens when you alternate reads and writes in a loop. Every read after a write forces the browser to recalculate layout immediately.

```js
// ❌ Forces a layout on every iteration
boxes.forEach(box => {
  box.style.width = box.parentElement.offsetWidth + 'px';
});

// ✅ Batch reads, then batch writes
const widths = boxes.map(box => box.parentElement.offsetWidth);
requestAnimationFrame(() => {
  boxes.forEach((box, i) => { box.style.width = widths[i] + 'px'; });
});
```

Reading `offsetWidth`, `getBoundingClientRect()`, `scrollTop` or `getComputedStyle()` can all force layout.

**Useful tools**
- **`will-change: transform`** promotes an element to its own layer. Use it sparingly, since every layer costs memory.
- **`content-visibility: auto`** skips rendering off-screen sections.
- **`contain: layout paint`** limits how far a change can spread.

**Scenario:** *"A sidebar animation stutters on low-end phones."*
It's animating `left` or `width`, which triggers layout on every frame. Switch to `transform: translateX()`. Confirm in the Chrome Performance tab: the purple Layout bars should disappear.

---

### 24.3 Event Propagation & Delegation

**Explanation**
A DOM event travels in **three phases**:

```
window → document → … → parent → TARGET → parent → … → document → window
   └──── 1. capture ────┘   2. target   └────── 3. bubble ──────┘
```

| API | Effect |
|---|---|
| `e.stopPropagation()` | Stops the event moving further along the path |
| `e.stopImmediatePropagation()` | Also stops other listeners on the same element |
| `e.preventDefault()` | Cancels the browser's default action (form submit, link navigation). It doesn't stop propagation. |
| `e.target` | The element the event started on |
| `e.currentTarget` | The element whose listener is running |

**Event delegation** means putting one listener on a parent instead of one on every child. It uses less memory and works for children added later.

```js
list.addEventListener('click', e => {
  const item = e.target.closest('[data-id]');
  if (item && list.contains(item)) select(item.dataset.id);
});
```

**In React:** handlers bubble by default, and `onClickCapture` listens in the capture phase. React already uses delegation at the root ([1.10](#110-synthetic-events--event-handling)), and events bubble through the React tree even from Portals ([5.6](#56-portals)).

**Passive listeners:** `{ passive: true }` promises you won't call `preventDefault()`, so the browser can scroll without waiting for your handler.

```js
window.addEventListener('scroll', onScroll, { passive: true });
```

**Scenario:** *"A 'click outside to close' dropdown closes as soon as it's opened."*
The click that opened it bubbles up to the `document` listener you just added, or a handler inside stops propagation so the outside check misbehaves. Listen on `pointerdown` in the capture phase, and check `!menuRef.current.contains(e.target)`.

---

### 24.4 Browser Storage

**Explanation**

| | Cookies | `localStorage` | `sessionStorage` | IndexedDB | Cache API |
|---|---|---|---|---|---|
| Capacity (approx.) | ~4 KB each | ~5 MB | ~5 MB | Hundreds of MB or more | Shares the origin's quota |
| Lifetime | Set by expiry | Until cleared | Until the tab closes | Until cleared | Until cleared |
| Sent to the server | **Every request** | No | No | No | No |
| API | String | Synchronous | Synchronous | Asynchronous | Asynchronous |
| Available in workers | No | No | No | **Yes** | **Yes** |
| Best for | Session IDs | Small preferences | Per-tab state | Offline data, large structured data | Offline assets and responses |

- `localStorage` is **synchronous**, so large reads block the main thread.
- Any JS on the page can read `localStorage`, so don't store auth tokens there ([17.3](#173-auth-token-storage-cookies-vs-localstorage)).
- It doesn't exist during SSR, so reading it during render causes crashes or hydration mismatches.
- Browsers can evict storage under pressure. Call `navigator.storage.persist()` when the data matters.

```js
// Sync across tabs: 'storage' fires in OTHER tabs only
window.addEventListener('storage', e => {
  if (e.key === 'session' && e.newValue === null) logoutLocally();
});

// Or send messages between tabs directly
const channel = new BroadcastChannel('auth');
channel.postMessage({ type: 'logout' });
channel.onmessage = e => { if (e.data.type === 'logout') logoutLocally(); };
```

**Scenario:** *"Logging out in one tab should log the user out of all tabs."*
Use `BroadcastChannel` or the `storage` event to notify other tabs, then clear their state and redirect. Also invalidate the session on the server, so a tab that missed the message can't keep working.

---

### 24.5 Browser APIs Worth Knowing

**Explanation**

| API | Purpose | Replaces |
|---|---|---|
| `IntersectionObserver` | Detect when an element enters the viewport | Scroll listeners with `getBoundingClientRect` |
| `ResizeObserver` | Detect an element's size change | Window `resize` listeners |
| `MutationObserver` | Watch for DOM changes | Polling |
| `requestAnimationFrame` | Run code just before the next paint | `setTimeout` for animation |
| `requestIdleCallback` | Run low-priority work when the browser is idle | Running it immediately |
| Page Visibility API | Detect a hidden tab | Polling that never stops |

```jsx
function useInView(ref) {
  const [inView, setInView] = useState(false);
  useEffect(() => {
    const observer = new IntersectionObserver(([entry]) => setInView(entry.isIntersecting));
    if (ref.current) observer.observe(ref.current);
    return () => observer.disconnect();
  }, [ref]);
  return inView;
}
```

**Why `requestAnimationFrame` beats `setTimeout` for animation:** it's aligned with the display's refresh, so frames aren't dropped or wasted, and it pauses in background tabs.

**Scenario:** *"A dashboard keeps polling and draining battery while the tab is in the background."*
Listen for `visibilitychange`, then stop polling when `document.hidden` is true and refresh when it's visible again. React Query does this by default with `refetchIntervalInBackground: false`.

---

### 24.6 Interview Questions

**Q1. What happens from typing a URL to seeing the page?** ⭐
DNS lookup, then a TCP and TLS connection, then the HTTP request. As the response arrives, the browser parses HTML into the DOM, fetches CSS and JS, builds the CSSOM, and combines them into a render tree. Then it runs layout, paint and composite. JS runs along the way, and in a React SPA it mounts the app and fills the page.

**Q2. What's the difference between reflow and repaint?** ⭐
Reflow recalculates geometry and is the most expensive step, because it can affect many elements. Repaint redraws pixels without changing geometry. `transform` and `opacity` can skip both and run on the compositor.

**Q3. Explain event bubbling, capturing and delegation.** ⭐
An event travels down from the window (capture), reaches the target, then goes back up (bubble). Listeners run in the bubble phase by default. Delegation puts one listener on a parent and uses `e.target` to find the child, which saves memory and handles dynamic children.

**Q4. `stopPropagation` vs `preventDefault`?** ⭐
`stopPropagation` stops the event reaching other elements. `preventDefault` cancels the browser's default action. They're independent, and you can use either or both.

**Q5. Cookies vs `localStorage` vs `sessionStorage`?** ⭐
Cookies are small, sent with every request, and can be `HttpOnly`, which makes them right for session IDs. `localStorage` persists and is larger, but it's readable by any script. `sessionStorage` is per tab and clears when the tab closes. Use IndexedDB for large or offline data.

**Q6. What is layout thrashing, and how do you avoid it?**
It's alternating DOM reads and writes, forcing a layout each time. Batch all reads first and all writes after, ideally in `requestAnimationFrame`.

**Q7. Why is CSS render-blocking?**
Painting without styles would flash unstyled content and then re-layout. So the browser waits for the CSSOM before the first paint. That's why critical CSS should be small and load early.

**Q8. Why does a heavy script make the page unresponsive?**
JS, layout, paint and input handling all share the main thread. A long task blocks everything else. Break up the work, move it to a Web Worker ([27.1](#271-web-workers)), or use `startTransition` for React renders.

---

## 25. Network & Data Fetching Fundamentals

### 25.1 Same-Origin Policy & CORS

**Explanation**
An **origin** is scheme + host + port. `https://app.com` and `https://api.app.com` are different origins.

The **same-origin policy** stops a page's JS from reading responses from another origin. **CORS** is how a server opts in to allowing that.

**Simple vs preflighted requests**
- **Simple requests** (GET, HEAD or POST with basic headers) are sent directly. The browser then checks the response headers.
- **Everything else** (such as PUT, DELETE, `Content-Type: application/json`, or `Authorization` headers) triggers a **preflight**: an `OPTIONS` request asking for permission first.

```
Browser ──OPTIONS /orders──────────────────► API
        Origin: https://app.com
        Access-Control-Request-Method: PUT

Browser ◄──204───────────────────────────── API
        Access-Control-Allow-Origin: https://app.com
        Access-Control-Allow-Methods: PUT
        Access-Control-Max-Age: 600          ← caches the preflight

Browser ──PUT /orders (the real request)───► API
```

**Sending cookies cross-origin**

```js
fetch('https://api.app.com/me', { credentials: 'include' });
```

The server must respond with `Access-Control-Allow-Credentials: true` and an **exact origin**. `Access-Control-Allow-Origin: *` isn't allowed with credentials.

**Three things people get wrong**
- CORS is enforced **by the browser**. `curl` and server-to-server calls ignore it.
- **CORS doesn't protect your API.** It only controls which web pages can read responses, so you still need authentication.
- The fix is on the **server**. The frontend can't fix a CORS error by itself.

**In development**, proxy API calls so the browser sees one origin:

```js
// vite.config.js
export default { server: { proxy: { '/api': 'http://localhost:8000' } } };
```

**Scenario:** *"The API works in Postman but fails from the browser with a CORS error."*
Postman doesn't enforce CORS. Check the Network tab for a failed `OPTIONS` request. The server must allow your origin, method and headers. A very common cause is an auth layer rejecting `OPTIONS` because preflights don't carry credentials.

---

### 25.2 HTTP Caching

**Explanation**
Browsers and CDNs can reuse responses, so the fastest request is the one that's never sent.

| Header | Meaning |
|---|---|
| `Cache-Control: max-age=3600` | Fresh for an hour, with no request needed |
| `no-cache` | Can be stored, but **must revalidate** before use |
| `no-store` | Never stored (sensitive data) |
| `public` / `private` | Shared caches allowed / browser only |
| `immutable` | Never changes, so don't revalidate |
| `stale-while-revalidate=60` | Serve stale for 60s while refreshing in the background |
| `ETag` / `If-None-Match` | Version check. If unchanged, the server returns `304` with no body. |

**`no-cache` doesn't mean "don't cache"**. That's `no-store`.

**The standard setup for a React app**

```
index.html              Cache-Control: no-cache                          ← always check for a new version
/assets/app.3f9a1c.js   Cache-Control: public, max-age=31536000, immutable  ← hash in the name = safe forever
```

A new deploy produces new file names, so `index.html` points to them and users get the update with no manual cache-busting.

**Scenario:** *"After a deploy, some users get a white screen with `ChunkLoadError`."*
Their open tab references old chunks that were deleted. Keep old assets available for a while after each deploy. Also catch the failed dynamic import in an error boundary and prompt the user to reload.

---

### 25.3 REST vs GraphQL on the Client

**Explanation**

| | REST | GraphQL |
|---|---|---|
| Endpoints | One per resource | Usually a single endpoint |
| Data shape | Fixed by the server | Chosen by the client |
| Over-fetching | Common | Avoided |
| Under-fetching | Multiple round trips | One request for nested data |
| HTTP caching | Works naturally | Harder (usually POST), so client-side caches matter more |
| Types | Through OpenAPI | Built into the schema, easy to generate TypeScript |
| Risk | Chatty screens | Expensive queries, N+1 on the server |

**Apollo Client** keeps a **normalised cache**: objects are stored once by `__typename` and `id`. Updating one object updates every component showing it.

```jsx
const GET_POST = gql`
  query GetPost($id: ID!) {
    post(id: $id) { id title author { id name } }
  }
`;

function Post({ id }) {
  const { data, loading, error } = useQuery(GET_POST, { variables: { id } });
  if (loading) return <Spinner />;
  if (error) return <ErrorMessage error={error} />;
  return <h1>{data.post.title}</h1>;
}
```

**React Query vs Apollo:** Apollo's normalised cache updates shared entities automatically but is GraphQL-specific. React Query caches by query key, works with any async function (including GraphQL), and is simpler, though you invalidate related queries yourself. Other GraphQL clients include urql and Relay.

**Scenario:** *"A mobile screen makes 6 REST calls and loads slowly on 3G."*
Options are a BFF endpoint that aggregates them, or GraphQL, where one query fetches the exact fields. Either way, the expensive round trips move to the fast server network.

---

### 25.4 Making Data Fetching Robust

**Explanation**
**`fetch` doesn't reject on 4xx or 5xx.** It only rejects on network failure, so always check `response.ok`.

```js
async function api(url, options) {
  const res = await fetch(url, { signal: AbortSignal.timeout(10_000), ...options });
  if (!res.ok) throw new ApiError(res.status, await res.text());
  return res.json();
}
```

| | `fetch` | axios |
|---|---|---|
| Rejects on 4xx/5xx | No | Yes |
| JSON parsing | Manual | Automatic |
| Interceptors | Write your own wrapper | Built in |
| Bundle size | None (built in) | Extra dependency |

**Refreshing an expired token:** when many requests get a `401` at once, send only **one** refresh call and have the others wait for it.

```js
let refreshing = null;

async function authFetch(url, options) {
  let res = await fetch(url, options);
  if (res.status !== 401) return res;

  refreshing ??= refreshToken().finally(() => { refreshing = null; });
  await refreshing;                  // every request shares the same promise
  return fetch(url, options);
}
```

**Also consider**
- **Retries** with exponential backoff, only on network errors, 429 and 5xx. Don't retry a POST unless it's idempotent.
- **Request de-duplication**, which React Query does for you.
- **Cancellation** with `AbortController` ([3.2](#32-useeffect)).
- **Prefetching** on hover or in route loaders.

**Scenario:** *"When a token expires, the page fires 8 refresh calls and logs the user out."*
With rotating refresh tokens, the second refresh uses a token that's already been spent. Share a single in-flight promise as shown above.

---

### 25.5 Interview Questions

**Q1. What is CORS, and why does it exist?** ⭐
The same-origin policy stops a malicious page from reading your data on another site using your cookies. CORS lets a server explicitly allow specific origins to read its responses. It's enforced by browsers and configured on the server.

**Q2. What is a preflight request?** ⭐
An automatic `OPTIONS` request the browser sends before a non-simple request (such as a PUT, or JSON with custom headers) to ask whether it's allowed. `Access-Control-Max-Age` caches the answer and removes the extra round trip.

**Q3. What's the difference between `no-cache` and `no-store`?** ⭐
`no-cache` stores the response but revalidates before each use, often getting a cheap `304`. `no-store` never stores it. Use `no-store` for sensitive data.

**Q4. How do you cache a React app's assets so users always get the latest version?** ⭐
Give JS and CSS content-hashed file names with a one-year `immutable` cache, and serve `index.html` with `no-cache`. Each deploy changes the file names, so one small HTML check is enough to pick up the new version.

**Q5. REST or GraphQL for a new project?** ⭐
REST for simple resources, public APIs and easy HTTP caching. GraphQL when many clients need different shapes of nested data and you can afford the server complexity. For a small team with one client, REST with React Query is usually enough.

**Q6. Does `fetch` throw on a 404?** ⭐
No. It resolves with `ok: false`. It only rejects on network errors or aborts, so you have to check `response.ok` yourself.

**Q7. What is an ETag?**
A version identifier for a response. The browser sends it back in `If-None-Match`, and if nothing changed the server replies `304 Not Modified` with no body. You still make a request, but you skip the download.

**Q8. How does a normalised cache work?**
Each object is stored once, keyed by type and ID, and queries hold references to it. Updating an object in one place updates everything using it. The cost is more complexity in cache configuration and pagination.

---

## 26. Memory Leaks

### 26.1 What Causes Memory Leaks in React Apps

**Explanation**
JavaScript is garbage collected: memory is freed when an object can no longer be **reached**. A leak happens when something long-lived keeps a reference to something you've finished with. Single-page apps are especially exposed, because the page never reloads to clear memory.

| Cause | Example | Fix |
|---|---|---|
| Event listeners not removed | `window.addEventListener` in an Effect | Remove it in the cleanup |
| Timers still running | `setInterval` with no `clearInterval` | Clear it in the cleanup |
| Open subscriptions | WebSocket, store or observer subscriptions | Unsubscribe or `disconnect()` in the cleanup |
| Unbounded caches | Module-level `Map` that only ever grows | Add a size limit or TTL (LRU) |
| Closures holding large data | A long-lived callback capturing a big array | Capture only what you need |
| Detached DOM nodes | A ref or global holding a removed element | Clear the reference |
| Third-party instances | Chart or map libraries not destroyed | Call `.destroy()` in the cleanup |
| Object URLs | `URL.createObjectURL` without a revoke | Call `URL.revokeObjectURL` |

```jsx
// ❌ Leaks: each mount adds a listener, and the component stays reachable
useEffect(() => {
  window.addEventListener('resize', onResize);
}, []);

// ✅ Every setup has a matching cleanup
useEffect(() => {
  window.addEventListener('resize', onResize);
  const id = setInterval(poll, 5000);
  const chart = createChart(ref.current);
  return () => {
    window.removeEventListener('resize', onResize);
    clearInterval(id);
    chart.destroy();
  };
}, []);
```

**A common misconception:** older React versions warned "Can't perform a state update on an unmounted component". React 18 removed that warning, because most such cases weren't real leaks. A pending fetch finishing after unmount is harmless. A subscription that's never cleaned up **is** a leak.

`removeEventListener` needs the **same function reference**, so a new inline arrow function won't remove anything.

**Scenario:** *"Our dashboard gets slower the longer it stays open, and the tab eventually crashes."*
The cause is usually a live data feed pushing into an array that's never trimmed, or chart instances created on every update without being destroyed. Cap the stored data (for example, the last 1,000 points), reuse the chart instance, and confirm with heap snapshots as shown next.

---

### 26.2 Finding Leaks with Chrome DevTools

**Explanation**
Don't guess. Measure with a repeatable action.

**The three-snapshot method**
1. Open **DevTools → Memory** and take a **heap snapshot** (this triggers garbage collection first).
2. Do the suspected action several times, such as opening and closing a modal 10 times.
3. Take a second snapshot and repeat the action.
4. Take a third snapshot and use the **Comparison** view. Objects that keep growing are suspects.
5. Click an object and read its **Retainers** panel, which shows what's keeping it alive.

| Tool | What it shows |
|---|---|
| **Heap snapshot** | What's in memory, and who's holding it |
| **Allocation instrumentation on timeline** | Which code allocates memory that's never freed |
| **Performance Monitor** | Live JS heap size, DOM node count and listener count |
| **Performance tab (with Memory enabled)** | Memory over time |

**Reading the results**
- A **sawtooth** pattern (memory rises, then drops at each GC) is healthy.
- A **rising baseline** after each GC suggests a leak.
- In a snapshot, filter by **"Detached"** to find DOM nodes removed from the page but still in memory.

**Avoiding false alarms:** test a production build where you can. In development, StrictMode double-invokes Effects, and DevTools or console logs can keep objects alive by themselves.

**Scenario:** *"How do you prove a leak is fixed?"*
Run the same repeated action before and after the fix, and compare. The object count and DOM node count should return to the baseline after GC. You can automate it in a Playwright test by measuring heap size across repeated navigation.

---

### 26.3 `WeakMap`, `WeakSet` & `WeakRef`

**Explanation**
Weak references don't stop an object being garbage collected.

| | Holds its keys or target | Use for |
|---|---|---|
| `Map` | Strongly: stays in memory until removed | Normal key-value data |
| `WeakMap` | Weakly: the entry disappears once the key object is unreachable | Attaching data to objects you don't own, such as per-element metadata |
| `WeakSet` | Weakly | Marking objects as seen |
| `WeakRef` | Weakly: `.deref()` may return `undefined` | Optional caches of large objects |

```js
const metadata = new WeakMap();

function track(element) {
  metadata.set(element, { firstSeen: Date.now() });
}
// When the element is gone, its metadata can be collected too. No manual cleanup.
```

`WeakMap` keys must be objects (or non-registered symbols), and it can't be iterated. Garbage collection timing isn't predictable, so never put program logic on it.

**Scenario:** *"We cache computed data per object, and the cache keeps growing."*
If the cache is keyed by object, a `WeakMap` drops each entry when the object goes away. If it's keyed by strings, weak references don't apply, so use an LRU cache with a size limit.

---

### 26.4 Interview Questions

**Q1. What is a memory leak in JavaScript?** ⭐
Memory that's no longer needed but stays reachable, so the garbage collector can't free it. In a long-running SPA, it builds up until the page slows or the tab crashes.

**Q2. What are common causes of memory leaks in React?** ⭐
Effects without cleanup (listeners, timers, subscriptions), unbounded module-level caches, third-party instances that are never destroyed, closures capturing large data, and refs or globals holding detached DOM nodes.

**Q3. How would you debug a memory leak?** ⭐
Reproduce it with a repeatable action. Take heap snapshots before and after repeating it, then use the Comparison view to find growing objects. Follow the Retainers panel to see what's holding them. Filter for detached nodes, fix the cause, and repeat the test to confirm.

**Q4. Is calling `setState` after unmount a memory leak?** ⭐
Usually not, which is why React 18 removed the warning. A finished fetch has no lasting reference, so it's just a wasted update. A subscription that keeps calling `setState` forever is the real leak, and that needs cleanup.

**Q5. What are detached DOM nodes?**
Elements removed from the document that are still referenced from JS, such as by a variable, a closure or an event listener. They stay in memory along with their children. Filter for "Detached" in a heap snapshot to find them.

**Q6. When would you use a `WeakMap`?**
To associate data with an object without keeping that object alive, such as caching per-DOM-node data or private per-instance data. The entry is collected along with the key, so no manual cleanup is needed.

---

## 27. Web Workers, Service Workers & PWA

### 27.1 Web Workers

**Explanation**
JavaScript runs on one main thread, which also handles rendering and input. A **Web Worker** runs JS on a **background thread**, so heavy work doesn't freeze the UI.

- **No DOM access**: no `document` or `window`, and no React.
- **Communication** is through `postMessage`. Data is **copied** using the structured clone algorithm.
- **Transferable objects** such as `ArrayBuffer` move ownership with no copy, which is much faster for large data.

```js
// worker.js
self.onmessage = e => {
  const result = parseLargeCsv(e.data);
  self.postMessage(result);
};
```

```jsx
function useCsvParser() {
  const [rows, setRows] = useState(null);
  const workerRef = useRef(null);

  useEffect(() => {
    const worker = new Worker(new URL('./worker.js', import.meta.url), { type: 'module' });
    worker.onmessage = e => setRows(e.data);
    workerRef.current = worker;
    return () => worker.terminate();        // don't leak the thread
  }, []);

  return { rows, parse: text => workerRef.current.postMessage(text) };
}
```

**Good uses:** parsing large files, image processing, encryption, search indexes, heavy calculations. **Poor uses:** small tasks, because copying data and starting a worker has its own cost. Libraries like **Comlink** make a worker look like an ordinary async function.

**Web Worker vs `startTransition`:** `startTransition` keeps the UI responsive during heavy **React rendering**, but the work still happens on the main thread. A worker moves heavy **non-React computation** off the main thread entirely.

**Scenario:** *"Importing a 50 MB CSV freezes the page for 10 seconds."*
Parsing blocks the main thread, so nothing can repaint or respond. Move it to a Web Worker, stream the file in chunks, post progress messages, and virtualise the results table.

---

### 27.2 Service Workers

**Explanation**
A **Service Worker** is a background script that acts as a **programmable network proxy** between your app and the network. It lets you control caching and provide offline support. It requires HTTPS (except on localhost) and has no DOM access.

**Lifecycle:** `register` → `install` (pre-cache assets) → `waiting` → `activate` (clean up old caches) → `fetch` (intercept requests).

```js
// sw.js
const CACHE = 'app-v2';

self.addEventListener('install', e => {
  e.waitUntil(caches.open(CACHE).then(c => c.addAll(['/', '/offline.html'])));
});

self.addEventListener('activate', e => {
  e.waitUntil(caches.keys().then(keys =>
    Promise.all(keys.filter(k => k !== CACHE).map(k => caches.delete(k)))
  ));
});

self.addEventListener('fetch', e => {
  if (e.request.mode !== 'navigate') return;
  e.respondWith(fetch(e.request).catch(() => caches.match('/offline.html')));
});
```

**Caching strategies**

| Strategy | How | Use for |
|---|---|---|
| **Cache first** | Cache, then network on a miss | Hashed assets, fonts |
| **Network first** | Network, then cache if offline | HTML, frequently changing data |
| **Stale-while-revalidate** | Serve from cache, update in the background | Avatars, non-critical API data |
| **Network only** | Never cached | Payments, sensitive data |

**The update trap:** a new Service Worker waits until **all tabs running the old one are closed**. That's why users can stay on an old version. Common fixes are showing an "Update available, refresh" prompt, or calling `skipWaiting()` with care, because mixing a new worker with an already-loaded page can break it.

In practice most teams use **Workbox** (or `vite-plugin-pwa`) rather than writing this by hand.

**Scenario:** *"We fixed a bug, but users still see the old version days later."*
A cache-first Service Worker keeps serving the old `index.html`. Use network-first for HTML, show a refresh prompt when a new worker is waiting, and serve the `sw.js` file itself with `no-cache`. Keep a "kill switch" worker ready, which removes the caches and unregisters.

---

### 27.3 Progressive Web Apps & Offline-First

**Explanation**
A **PWA** is a website that behaves like an installed app. It needs **HTTPS**, a **web app manifest**, and usually a Service Worker.

```json
{
  "name": "Field Sales",
  "short_name": "Sales",
  "start_url": "/",
  "display": "standalone",
  "theme_color": "#0b5fff",
  "icons": [{ "src": "/icon-512.png", "sizes": "512x512", "type": "image/png" }]
}
```

**Capabilities:** install to the home screen, offline use, web push notifications, and background sync on supported browsers. Support varies, especially on iOS Safari, so check before relying on a feature.

**Offline-first** means the app reads and writes locally first, then syncs.

```
UI ──► IndexedDB (source of truth for the UI)
           │ pending changes go into an outbox
           ▼
   online? ──► send to the server with idempotency keys ──► mark as synced
```

Decisions to cover in an interview:
- **Conflict resolution** when two devices edit the same item: last-write-wins, field-level merge, or asking the user.
- **Idempotency keys**, so a retried sync can't create duplicates.
- **Showing sync state** in the UI ("Saved offline", "Syncing…").

**Scenario:** *"Delivery agents need to record drop-offs in areas with no signal."*
Save each drop-off to IndexedDB immediately and add it to an outbox. Sync when the connection returns, using background sync where it's available and a retry on app start otherwise. Pre-cache the app shell with a Service Worker so it opens with no signal.

---

### 27.4 Interview Questions

**Q1. Web Worker vs Service Worker?** ⭐
A Web Worker is a background thread for heavy computation, tied to the page that created it. A Service Worker is a network proxy for caching and offline support, and it's event-driven and can outlive the page. Neither has DOM access.

**Q2. How do you keep the UI responsive during heavy computation?** ⭐
Move it to a Web Worker. If that isn't practical, break the work into chunks and yield to the main thread between them. If the slow part is React rendering rather than computation, use `startTransition`, `useDeferredValue` or virtualisation.

**Q3. Explain the Service Worker lifecycle.** ⭐
Register, then install (usually pre-caching assets), then wait until old versions have no open tabs, then activate (cleaning old caches), then handle `fetch` events. The waiting step exists so one page never gets two different workers mid-session.

**Q4. Which caching strategies would you use for which resources?** ⭐
Cache first for hashed static assets, network first for HTML and important data, stale-while-revalidate for content that can be slightly old, and network only for sensitive or transactional requests.

**Q5. What makes a website a PWA?** ⭐
HTTPS, a web app manifest and, in practice, a Service Worker for offline support. Together they make it installable, and it can work offline and use push notifications.

**Q6. How would you handle offline edits and conflicts?**
Save changes locally in IndexedDB, queue them in an outbox, and sync with idempotency keys when online. Choose a conflict rule per data type, such as last-write-wins for simple fields or a merge for collaborative data. Show pending state clearly.

**Q7. What are transferable objects?**
Objects like `ArrayBuffer` whose ownership moves to the worker instead of being copied. The sender loses access, but transferring is nearly instant, even for large data.

---

## 28. Practical Features

### 28.1 File Uploads

**Explanation**
**Basic upload** with a file input and `FormData`. Let the browser set the `Content-Type` header, because it has to add the multipart boundary.

```jsx
function AvatarUpload() {
  const [preview, setPreview] = useState(null);

  useEffect(() => () => { if (preview) URL.revokeObjectURL(preview); }, [preview]);

  function onChange(e) {
    const file = e.target.files[0];
    if (!file) return;
    if (!file.type.startsWith('image/')) return alert('Images only');
    if (file.size > 5 * 1024 * 1024) return alert('Max 5 MB');
    setPreview(URL.createObjectURL(file));       // instant local preview
  }

  return (
    <>
      <input type="file" accept="image/*" onChange={onChange} />
      {preview && <img src={preview} alt="Preview" />}
    </>
  );
}
```

**Presigned URLs:** for anything large, upload **directly to object storage**, so the file doesn't pass through your servers.

```
1. Client ── POST /uploads {name, size, type} ──► API (checks auth and limits)
2. API ── returns a short-lived presigned URL ──► Client
3. Client ── PUT file ──► S3 / object storage
4. Client ── POST /uploads/:id/complete ──► API (saves metadata, starts processing)
```

**Progress:** `XMLHttpRequest` (and axios, which uses it in browsers) has upload progress events, while `fetch` doesn't offer a simple equivalent.

```js
function upload(url, file, onProgress, signal) {
  return new Promise((resolve, reject) => {
    const xhr = new XMLHttpRequest();
    xhr.upload.onprogress = e => e.lengthComputable && onProgress(e.loaded / e.total);
    xhr.onload = () => (xhr.status < 300 ? resolve() : reject(new Error(xhr.statusText)));
    xhr.onerror = () => reject(new Error('Network error'));
    signal?.addEventListener('abort', () => xhr.abort());
    xhr.open('PUT', url);
    xhr.send(file);
  });
}
```

**Large files:** split with `file.slice()` and upload chunks (S3 multipart or the **tus** protocol). Retry only failed chunks, upload a few in parallel, and store progress so an upload can resume after a refresh.

**Client validation is only for UX.** The server must re-check type, size and content, and scan for malware. File extensions and `file.type` can be faked.

**Scenario:** *"Users upload 2 GB videos on unreliable networks, and a failure at 95% restarts everything."*
Use resumable chunked uploads: split into 10 MB parts, send several in parallel, retry failed parts with backoff, and record completed parts so the upload can continue. Upload directly to storage with presigned URLs.

---

### 28.2 Animation

**Explanation**

| Approach | Best for |
|---|---|
| **CSS transitions** | Simple state changes such as hover and open/close |
| **CSS keyframes** | Looping or fixed sequences such as spinners |
| **`<ViewTransition>`** ([9.7](#97-react-193-view-transitions-fragment-refs--browser)) | Page and state changes, shared-element animation |
| **Motion** (formerly Framer Motion) | Gestures, exit animations, layout animation, springs |
| **Web Animations API** | Controlling animations from JS without a library |
| **Canvas / WebGL** | Games and heavy visuals |

**Performance:** animate `transform` and `opacity`, which stay on the compositor ([24.2](#242-reflow-repaint--compositing)). Avoid animating `width`, `height` or `top`.

**Exit animations are the hard part in React**, because an unmounted element is removed immediately. Libraries solve this by keeping the element until the animation finishes.

```jsx
import { AnimatePresence, motion } from 'motion/react';

<AnimatePresence>
  {isOpen && (
    <motion.div
      key="panel"
      initial={{ opacity: 0, y: 20 }}
      animate={{ opacity: 1, y: 0 }}
      exit={{ opacity: 0, y: 20 }}
    />
  )}
</AnimatePresence>
```

**FLIP** is the technique behind smooth layout animation: record the **F**irst position, move to the **L**ast position, **I**nvert with a `transform`, then **P**lay by removing it. This animates a layout change using only `transform`.

**Accessibility:** always respect the user's motion setting.

```css
@media (prefers-reduced-motion: reduce) {
  * { animation-duration: 0.01ms !important; transition-duration: 0.01ms !important; }
}
```

**Scenario:** *"An expanding accordion is janky because it animates `height`."*
`height` triggers layout on every frame. Options: CSS grid with `grid-template-rows` going from `0fr` to `1fr` (simple, though it still does layout), or a FLIP-style `transform` animation for large content. Check the result in the Performance tab.

---

### 28.3 Drag & Drop

**Explanation**

| Option | Pros | Cons |
|---|---|---|
| **HTML5 Drag and Drop API** | Built in, good for files from the desktop | No touch support, limited styling, poor keyboard access |
| **Pointer Events** | Works for mouse, touch and pen | You build everything yourself |
| **dnd-kit** | Mouse, touch and keyboard support, accessible, works with virtualised lists | Library to learn |

**Drop zone for files** (the native API is the right tool here):

```jsx
function DropZone({ onFiles }) {
  const [over, setOver] = useState(false);
  return (
    <div
      className={over ? 'drop active' : 'drop'}
      onDragOver={e => { e.preventDefault(); setOver(true); }}   // required, or drop won't fire
      onDragLeave={() => setOver(false)}
      onDrop={e => { e.preventDefault(); setOver(false); onFiles([...e.dataTransfer.files]); }}
    >
      Drop files here
    </div>
  );
}
```

**Sortable lists with dnd-kit:**

```jsx
<DndContext onDragEnd={({ active, over }) => {
  if (over && active.id !== over.id) {
    setItems(items => arrayMove(items, items.indexOf(active.id), items.indexOf(over.id)));
  }
}}>
  <SortableContext items={items}>
    {items.map(id => <SortableCard key={id} id={id} />)}
  </SortableContext>
</DndContext>
```

**Design points**
- **State shape:** keep IDs in order per column and items in a lookup: `{ columns: { todo: ['a', 'b'] }, tasks: { a: {...} } }`.
- **Optimistic updates:** move the card immediately, then roll back if the save fails.
- **Saving order:** a position like `1.5` between `1` and `2` (**fractional indexing**) updates a single row instead of renumbering the list.
- **Accessibility:** provide keyboard dragging and announce moves through a live region. A "Move to…" menu is a good fallback.

**Scenario:** *"Reordering a 500-card Kanban board is laggy, and the order sometimes resets."*
Every drag event re-renders the whole board. Memoise cards, keep drag position out of global state, and virtualise long columns. The reset happens when a slower, older save response overwrites a newer one: cancel stale requests or check a version number, and use fractional indexing so saves are small.

---

### 28.4 Interview Questions

**Q1. How would you build a file upload with progress and cancel?** ⭐
Validate on the client, get a presigned URL, and upload directly to storage with `XMLHttpRequest` or axios for progress events. Cancel with `abort()` or an `AbortController`. Handle failures with retries, and confirm completion with the API so it can save metadata.

**Q2. How do you handle very large file uploads?** ⭐
Chunk the file, upload parts in parallel, retry only failed parts, and track completed parts so uploads can resume. Upload straight to object storage rather than through app servers, and run processing asynchronously.

**Q3. Why is `transform` better than `left` for animation?** ⭐
`left` changes layout, so the browser recalculates and repaints on each frame on the main thread. `transform` can be handled by the compositor on the GPU, so it stays smooth even when the main thread is busy.

**Q4. How do you animate a component as it unmounts?** ⭐
React removes it immediately, so the element must be kept until the animation ends. Use `AnimatePresence` from Motion, `<ViewTransition>` inside a Transition, or a small state machine that plays the animation and unmounts on `transitionend`.

**Q5. Why would you use dnd-kit rather than the HTML5 API?** ⭐
The HTML5 API doesn't work on touch devices, is hard to style and has weak keyboard support. dnd-kit uses pointer events, supports keyboards and screen readers, and gives you full rendering control. The native API remains a good fit for dropping files from the desktop.

**Q6. How do you persist the order of a reorderable list efficiently?**
Fractional indexing gives the moved item a position between its neighbours, so only one row changes. Renumbering everything is simple but costs a large write on every move. Occasionally rebalance positions if the fractions get too long.

**Q7. Why must you call `URL.revokeObjectURL`?**
Each object URL keeps the file in memory until the page unloads. Without revoking, repeated previews leak memory ([26.1](#261-what-causes-memory-leaks-in-react-apps)).

---

## 29. Frontend System Design Case Studies

Each case study uses the **RADIO** framework from [19.2](#192-frontend-system-design-the-framework): **R**equirements, **A**rchitecture, **D**ata model, **I**nterface (API), **O**ptimisations. In a 45-minute round, spend about 5 minutes on requirements and most of your time on the hard parts.

### 29.1 Design an Autocomplete / Typeahead

**Requirements**
- *Functional:* suggestions as the user types, keyboard navigation, select an item, recent searches.
- *Non-functional:* feels instant (under ~100 ms for cached results), low network use, accessible, reusable across products.

**Architecture**

```
<Autocomplete>
 ├─ <Input>            value, keyboard events, ARIA combobox
 ├─ useSuggestions()   debounce → cache → fetch → cancel stale requests
 └─ <Listbox>          renders results (Portal if inside overflow:hidden), highlights matches
```

**Data model**

```ts
type State = {
  query: string;
  results: Suggestion[];
  status: 'idle' | 'loading' | 'success' | 'error';
  activeIndex: number;
  isOpen: boolean;
};
const cache = new Map<string, { results: Suggestion[]; at: number }>();   // LRU with a TTL
```

**Interface:** `GET /suggest?q=iph&limit=8`. Props: `fetchSuggestions`, `renderItem`, `onSelect`, `minChars`, `debounceMs`.

**Optimisations**
- **Debounce** at about 200 ms, with a minimum of 2 characters.
- **Cancel stale requests** with `AbortController`, so late responses can't overwrite newer ones.
- **Cache** results by query, including empty results.
- **Reuse earlier results:** filter locally when a previous result set is complete.
- **Accessibility:** `role="combobox"`, `aria-expanded`, `aria-activedescendant`, a `role="listbox"` popup, and a live region for the result count.
- **Security:** render highlights as React elements, never with `dangerouslySetInnerHTML`.

See [21.1](#211-debounced-autocomplete-with-keyboard-navigation-) for a working implementation.

**Interview questions**

**Q1. How do you stop old responses overwriting new ones?** ⭐
Abort the previous request with `AbortController` when the query changes, or compare a request ID before applying the result. Aborting also saves bandwidth.

**Q2. Debounce or throttle here?** ⭐
Debounce. You want one request after the user pauses, not regular requests while they type.

**Q3. How would you design the cache?** ⭐
A `Map` keyed by the normalised query with a size limit (LRU) and a short TTL, because suggestions change. Serve from cache instantly, and optionally refresh in the background.

**Q4. How do you make it accessible?** ⭐
Use the ARIA combobox pattern. Keep focus on the input and point `aria-activedescendant` at the highlighted option. Support arrow keys, Enter and Escape, and announce the number of results.

**Q5. How would you make it reusable across different products?**
Separate logic from UI: a headless `useCombobox` hook handles state, keyboard and ARIA, and the data source and item rendering are passed in as props. Teams keep their own styling while sharing the behaviour.

---

### 29.2 Design a Chat Application

**Requirements**
- *Functional:* conversations list, send and receive in real time, history, typing indicators, read receipts, attachments.
- *Non-functional:* instant-feeling sends, smooth scrolling through long histories, resilience to flaky networks, works across tabs and devices.

**Architecture**

```
<ChatApp>
 ├─ <ConversationList>   virtualised, sorted by last message
 └─ <ChatWindow>
     ├─ <MessageList>    reverse virtualised list, loads history on scroll up
     └─ <Composer>       draft saved per conversation, upload progress

Connection manager (outside React) ── WebSocket ── updates ──► normalised store
```

Keep the WebSocket in a **singleton outside the component tree**, so re-renders and route changes don't reconnect it.

**Data model**

```ts
type Message = {
  id: string;                  // server ID
  clientId: string;            // generated on the client, used to match optimistic messages
  conversationId: string;
  senderId: string;
  body: string;
  seq: number;                 // server order
  status: 'sending' | 'sent' | 'delivered' | 'read' | 'failed';
};
// Normalised: messagesById + messageIdsByConversation
```

**Interface**
- WebSocket events: `message.send`, `message.new`, `message.ack`, `typing`, `read`.
- REST: `GET /conversations/:id/messages?before=<cursor>&limit=50`.

**Optimisations**
- **Optimistic send:** show the message immediately as `sending`, then replace it when the ack with the matching `clientId` arrives.
- **Reliability:** reconnect with exponential backoff and jitter. On reconnect, fetch messages after the last known `seq`, and retry unsent messages from an outbox.
- **Scrolling:** a virtualised list, keeping scroll position when older messages load above, and only auto-scrolling when the user is already at the bottom. Show a "New messages" button otherwise.
- **Throttle typing events** to one every few seconds.
- **Offline:** store recent conversations and the outbox in IndexedDB.

**Interview questions**

**Q1. WebSocket, SSE or polling for chat?** ⭐
WebSocket, since messages, typing and receipts flow both ways with low latency. SSE works for one-way feeds, and long polling is only a fallback. See [19.3](#193-real-time-communication-websocket-vs-sse-vs-polling).

**Q2. How do optimistic messages work?** ⭐
The client generates a `clientId`, shows the message as `sending`, and sends it. The server's ack includes that `clientId`, so the client swaps in the final ID and marks it sent. A timeout marks it failed, with a retry option.

**Q3. How do you handle a dropped connection?** ⭐
Detect it with heartbeats, reconnect with backoff and jitter, and show a "reconnecting" banner. After reconnecting, fetch everything since the last `seq`, and resend the outbox. The server de-duplicates using `clientId`.

**Q4. How do you keep messages in order?** ⭐
Sort by a server-assigned sequence number, not client timestamps, because device clocks differ. A gap in `seq` tells the client it has to fetch the missing messages.

**Q5. How do you render 100,000 messages smoothly?**
Load pages on demand and virtualise so only visible rows exist in the DOM. Variable heights need measured rows. When prepending history, adjust the scroll offset so the view doesn't jump, and evict old pages from memory.

---

### 29.3 Design a Collaborative Editor (Google Docs)

**Requirements**
- *Functional:* rich text, several people editing at once, live cursors, comments, version history, offline edits.
- *Non-functional:* typing never waits for the network, all users converge to the same document, large documents stay fast.

**Architecture**

```
Editor UI (ProseMirror / Lexical / Slate)
     │ local edits apply immediately
     ▼
Document model + sync engine (CRDT like Yjs, or OT)
     │                         │
 IndexedDB (offline)    WebSocket ──► collaboration server ──► persistence
                               └─► presence channel (cursors, selections)
```

Don't build on plain `contentEditable` and React state. Use an editor framework with its own document model, because a re-render per keystroke won't scale, and browsers behave inconsistently.

**Data model**
- **Document:** a structured tree (paragraphs, headings, marks), not an HTML string.
- **Operations:** small changes such as insert, delete and format.
- **Presence:** kept separately from the document, because it's temporary and doesn't need saving.

**OT vs CRDT**

| | Operational Transformation | CRDT |
|---|---|---|
| How | The server transforms concurrent operations | Data structure merges automatically |
| Needs a central server | Usually | No |
| Offline | Harder | Natural fit |
| Cost | Complex transform logic | Extra metadata per character |
| Examples | Classic Google Docs approach | Yjs, Automerge |

**Interface:** WebSocket messages for `update` (binary), `awareness` (cursors) and `sync` (exchange missing changes on reconnect). REST for metadata, comments and version snapshots.

**Optimisations**
- **Batch updates** every 50–100 ms instead of per keystroke.
- **Throttle cursor updates**, and keep presence out of the undo history.
- **Snapshots:** periodically compact history so loading doesn't replay every edit.
- **Large documents:** virtualise or lazily render sections.
- **Undo:** per user, so you never undo someone else's edit.

**Interview questions**

**Q1. How do you resolve two users editing the same text at once?** ⭐
Use OT or a CRDT. With OT, the server adjusts operations against each other. With a CRDT, each character has a stable ID so changes merge the same way everywhere. For a new product, I'd choose a proven CRDT library like Yjs rather than writing my own.

**Q2. Why not use last-write-wins?** ⭐
Each save would overwrite the other person's work. Collaboration needs merging at the level of individual operations, which preserves everyone's intent.

**Q3. How does offline editing work?** ⭐
Edits apply locally and persist to IndexedDB. On reconnect, the client and server exchange missing updates, and the CRDT merges them. Both sides converge without manual conflict handling.

**Q4. How do you show other users' cursors?** ⭐
Through a separate, throttled presence channel. Cursors are stored as positions relative to the document model, so they stay correct as text changes. If a user disconnects, their presence expires.

**Q5. How does undo work with multiple users?**
Each user has their own undo stack, recording only their operations. Undo creates a new inverse operation rather than rewinding history, because others may have edited since. CRDT libraries such as Yjs provide this.

---

### 29.4 Design an Analytics Dashboard

**Requirements**
- *Functional:* configurable widgets, filters and date ranges, drill-down, export, near-real-time updates.
- *Non-functional:* fast first view, one slow widget can't block the others, handles large datasets, shareable views.

**Architecture**

```
<DashboardPage>                   URL holds the filters and date range
 ├─ <FilterBar>
 └─ <WidgetGrid>                  layout from saved config
     └─ <Widget>                  own <Suspense> + <ErrorBoundary>, lazy-loads its chart
         └─ useQuery(['widget', id, filters])
```

**Data model**

```ts
type Dashboard = { id: string; layout: GridItem[]; widgets: WidgetConfig[] };
type WidgetConfig = { id: string; type: 'line' | 'bar' | 'table' | 'kpi'; query: QuerySpec };
```

- **URL:** filters, date range and selected tab, so views can be shared ([11.5](#115-url-as-state-search-params)).
- **Server state:** React Query, keyed by widget and filters.
- **Local state:** hover and tooltips, kept in the widget.

**Interface:** `POST /query` per widget, or a batch endpoint. The server returns **pre-aggregated** data. Never send a million raw rows for the browser to aggregate. Exports run as a background job that returns a download link.

**Optimisations**
- **Independent loading:** per-widget Suspense and error boundaries, with a retry button on each.
- **Lazy loading:** load heavy chart libraries only when needed, and render widgets when they near the viewport.
- **Rendering:** SVG for small charts, Canvas or WebGL for many data points, and downsample to roughly the number of pixels available.
- **Tables:** virtualise, with server-side sorting and pagination.
- **Updates:** poll or use SSE, pause when the tab is hidden, and keep showing old data while refetching.
- **Heavy calculations:** move to a Web Worker ([27.1](#271-web-workers)).
- **Filters:** wrap expensive updates in `startTransition`.

**Interview questions**

**Q1. How do you stop one slow widget blocking the whole page?** ⭐
Give each widget its own data query, Suspense boundary and error boundary. They load and fail independently, and a skeleton holds the layout in place. Avoid one large API call for the whole dashboard.

**Q2. How would you render a chart with 1 million points?** ⭐
Aggregate or downsample on the server, because a screen can't show more points than pixels. Use Canvas or WebGL rather than SVG for large datasets, and fetch more detail as the user zooms.

**Q3. Where should filter state live?** ⭐
In the URL, so views can be shared, bookmarked and restored on refresh. Components read from the URL, and query keys include the filters so caching works automatically.

**Q4. How do you keep data fresh without overloading the backend?** ⭐
Poll at a sensible interval per widget, or use SSE for truly live data. Pause when the tab is hidden, don't start a new request while one is in flight, and add jitter so clients don't all hit the server together.

**Q5. How would you design user-customisable dashboards?**
Store layout and widget settings as JSON, and render through a registry mapping widget types to lazily loaded components. Version the schema so old dashboards can be migrated. Adding a widget type then doesn't change the dashboard code.

---

### 29.5 Design a Video Player (YouTube / Netflix)

**Requirements**
- *Functional:* play, pause, seek, volume, quality, captions, fullscreen, resume where you left off.
- *Non-functional:* fast start, minimal buffering on poor networks, accessible controls, works across devices.

**Architecture**

```
<VideoPlayer>
 ├─ <video>                  native element, ref held by usePlayer()
 ├─ usePlayer()              state machine: idle → loading → playing ⇄ paused ⇄ buffering → ended | error
 ├─ Streaming engine         hls.js / dash.js / Shaka via Media Source Extensions
 ├─ <Controls>               progress, volume, quality, captions, fullscreen
 └─ Analytics                startup time, rebuffer rate, errors
```

**Adaptive bitrate (ABR):** the video is split into short segments at several qualities, listed in a manifest (HLS or DASH). The player measures bandwidth and buffer level, and chooses the quality for each next segment.

**Data model**

```ts
type PlayerState = {
  status: 'idle' | 'loading' | 'playing' | 'paused' | 'buffering' | 'ended' | 'error';
  currentTime: number;
  buffered: TimeRange[];
  quality: 'auto' | '1080p' | '720p' | '480p';
};
```

**Interface:** `GET /videos/:id` returns metadata, manifest URL and caption tracks. `POST /videos/:id/progress` saves position every ~10 seconds and on page hide, using `navigator.sendBeacon` so it isn't lost when the tab closes.

**Optimisations**
- **Avoid per-frame React state:** `timeupdate` fires often. Update the progress bar through a ref, or keep the state inside `<Controls>` so the page doesn't re-render.
- **Fast start:** preconnect to the CDN, load the manifest early, begin at a low quality, and use `poster` for the first frame.
- **Lazy loading:** load the player library only when it's needed.
- **Feeds:** autoplay muted using `IntersectionObserver`, and pause when off-screen.
- **Accessibility:** captions with `<track>`, keyboard shortcuts, labelled controls, and Picture-in-Picture support.
- **Protected content:** DRM through Encrypted Media Extensions, plus signed URLs.

**Interview questions**

**Q1. How does adaptive streaming work?** ⭐
The video is cut into segments encoded at multiple bitrates. The player reads a manifest, watches throughput and buffer level, and picks the next segment's quality. That's why quality drops rather than stalling on a weak network.

**Q2. How do you stop the player re-rendering constantly?** ⭐
Keep frequently changing values like `currentTime` out of parent state. Update the progress bar directly through a ref or `requestAnimationFrame`, or isolate that state in a small child component.

**Q3. Why model the player as a state machine?** ⭐
Because states like buffering, seeking and errors interact. Explicit states and transitions prevent impossible combinations (such as "playing" and "error" together) and make the UI predictable.

**Q4. How would you save watch progress reliably?** ⭐
Send it periodically, and on `visibilitychange` when the page is hidden, using `sendBeacon` or `fetch` with `keepalive`, because normal requests can be cancelled when a tab closes. Debounce the saves to keep traffic low.

**Q5. Which metrics would you track?**
Startup time, rebuffer ratio, average bitrate, error rate and abandonment before playback begins. These connect the player's performance to user experience.

---

### 29.6 Design an E-commerce Site (Listing, Product Page, Checkout)

**Requirements**
- *Functional:* search and filter, product page, cart, checkout.
- *Non-functional:* excellent SEO, great Core Web Vitals on mobile, cart works across sessions, safe payments, handles sale traffic.

**Architecture: choose rendering per page**

| Page | Strategy | Why |
|---|---|---|
| Home, categories | SSG / ISR, or cached with `"use cache"` | SEO, rarely changes |
| Product page | Static shell + dynamic price and stock (streamed) | SEO with accurate data |
| Search results | SSR, with filters in the URL | Indexable and shareable |
| Cart, checkout | CSR or dynamic, never cached | Personal and sensitive |

**Data model**

```ts
type Cart = { id: string; items: { sku: string; qty: number }[]; version: number };
// The client holds SKUs and quantities. The server always calculates prices and totals.
```

- **URL:** filters, sort, page.
- **Server state:** products, stock and cart, through React Query or RSC.
- **Client state:** drawers and UI only.

**Interface:** `GET /products?category=shoes&cursor=…`, `GET /products/:id`, `POST /cart/items`, `POST /checkout` with an idempotency key.

**Optimisations**
- **LCP:** the main product image gets `fetchpriority="high"`, is served as AVIF or WebP with `srcset`, and is never lazy-loaded.
- **CLS:** reserve space with fixed image aspect ratios and skeletons.
- **INP:** keep client JS small with Server Components, and delay third-party scripts.
- **Cart:** optimistic updates, a cookie-identified guest cart, and merging into the account at login.
- **Checkout:** idempotency keys, a disabled submit button while processing, and hosted payment fields so card data never touches your code.
- **Sales:** CDN caching, a queue page for traffic spikes, and turning off non-essential features.
- **Analytics scripts:** manage them carefully, because they're often the biggest performance cost.

**Interview questions**

**Q1. Which rendering strategy would you use for each page?** ⭐
Static or ISR for catalogue pages because of SEO and speed. A static shell with streamed dynamic parts for product pages. Dynamic and uncached for cart and checkout, because they're personal. It's a per-route decision, not one app-wide choice.

**Q2. How do you show accurate stock and price on a cached product page?** ⭐
Cache the stable content (title, images, description) and fetch price and stock dynamically: a streamed Suspense boundary, or a client query. The server re-checks both at checkout, because displayed values can be out of date.

**Q3. How do you prevent duplicate orders when a user double-clicks Pay?** ⭐
Disable the button while submitting, but rely on an **idempotency key** generated once per checkout attempt. The server returns the original result for repeats, which also covers network retries.

**Q4. How do you handle guest carts?** ⭐
Identify the cart with a cookie and store it on the server (or in local storage for simple cases). At login, merge it into the user's cart with a clear rule for duplicates. Server storage keeps it working across sessions and devices.

**Q5. How would you improve a poor LCP on the product page?**
Find the LCP element first, usually the main image. Serve it from a CDN in a modern format and the right size, make it discoverable in the HTML, and set `fetchpriority="high"`. Render the page on the server, and cut render-blocking CSS and third-party scripts. Check with field data, not only Lighthouse.

---

### 29.7 Design a News Feed (Full Version)

This expands the summary in [19.2](#192-frontend-system-design-the-framework).

**Requirements**
- *Functional:* infinite feed of text, images and video, like, comment, compose, new post notifications.
- *Non-functional:* fast first load on 3G, smooth scrolling for thousands of posts, accessible, no layout jumps.

**Architecture**

```
Server render first page ──► hydrate ──► client takes over
<FeedPage>
 ├─ <Composer>                 optimistic posting
 ├─ <NewPostsBanner>           "12 new posts", never auto-inserted
 └─ <VirtualFeed>              variable-height virtual list
     └─ <PostCard>             memoised, reads by ID from a normalised store
```

**Data model**

```ts
type Store = {
  posts: Record<string, Post>;
  users: Record<string, User>;
  feed: { ids: string[]; nextCursor: string | null };
};
```

Normalising means a like on one post updates every place that post appears, and a change to one post only re-renders that card.

**Interface**
- `GET /feed?cursor=abc&limit=20` uses **cursor pagination**, because offset pagination duplicates or skips posts when new ones arrive.
- `POST /posts/:id/like`, with a real-time channel (SSE or WebSocket) for "new posts available".

**Optimisations**
- **Load more early:** an `IntersectionObserver` sentinel a few screens ahead, with prefetching.
- **Virtualise:** measure variable-height rows, and keep the DOM small.
- **Images:** responsive `srcset`, lazy loading after the first post, fixed aspect ratio, and a blurred placeholder.
- **Video:** autoplay muted in view, pause out of view, and only load one at a time.
- **Optimistic likes** with rollback on failure.
- **Restore scroll** on back navigation by keeping the cache and saved position.
- **Accessibility:** `role="feed"` with `role="article"`, and don't trap keyboard users in endless scrolling (keep the footer reachable, or add a "Load more" button).

**Interview questions**

**Q1. Why cursor pagination instead of offset?** ⭐
With offset, a new post shifts everything down, so the next page repeats an item. A cursor marks an exact position in the list, so it stays stable and is faster on the server for deep pages.

**Q2. Why show a "New posts" banner rather than inserting posts automatically?** ⭐
Inserting content above the viewport moves what the user is reading. The banner lets them choose when to load it, and the scroll position stays stable.

**Q3. How do you implement infinite scroll efficiently?** ⭐
An `IntersectionObserver` watches a sentinel near the bottom and triggers the next page, ideally before the user reaches the end. Virtualisation keeps the DOM small however many posts have loaded.

**Q4. How do you implement optimistic likes?** ⭐
Update the count immediately, send the request, and roll back with a message if it fails. React Query's `onMutate` and `onError`, or `useOptimistic`, handle this. Multiple quick taps should settle on a final state rather than sending one request per tap.

**Q5. How do you restore scroll position after opening a post and going back?**
Keep the feed in a cache (React Query or a store) so it isn't refetched, and save the scroll offset or the top visible post ID. With virtualisation, restore using the saved measurements or by scrolling to that item. `<Activity>` can also keep the feed mounted but hidden.

---

## Revision Cheat Sheet

**Doubts to revise**
-  Diffrence between Library and framework and package
-  JSX?
-  why component name should start with capital letter?
-  Should I copy props into state?
-  A child needs to change data shown in its sibling. How?"
-  "Users report that after deleting an item, the input text appears in the wrong row."
-  

| § | Topic | One-line takeaway |
|---|---|---|
| [1](#1-react-fundamentals) | Fundamentals | UI is a function of state; props in, state owned; stable keys; render must be pure |
| [1.8](#18-controlled-vs-uncontrolled-components) | Controlled vs uncontrolled | React holds the value vs the DOM holds it |
| [2.1](#21-virtual-dom--the-diffing-algorithm) | Virtual DOM | Compare descriptions, apply minimal changes; about simplicity more than raw speed |
| [2.2](#22-reconciliation-heuristics-position-type-key) | Reconciliation | Different type replaces the subtree; keys match list items |
| [2.3](#23-what-triggers-a-re-render) | Re-renders | State change, parent render, or Context change |
| [2.4](#24-render-phase-vs-commit-phase) | Render vs commit | Render is pure and interruptible; commit is synchronous |
| [2.6](#26-batching-automatic-batching-in-react-18) | Batching | Automatic everywhere since React 18 |
| [3.2](#32-useeffect) | `useEffect` | Synchronise with external systems; clean up; often you don't need one |
| [3.4](#34-uselayouteffect-vs-useeffect) | `useLayoutEffect` | Runs before paint; use to measure layout |
| [3.5](#35-useref) | `useRef` | Mutable value that doesn't re-render |
| [3.8](#38-usememo--usecallback) | `useMemo`/`useCallback` | Cache a value / a function; only when it helps |
| [3.10](#310-rules-of-hooks-and-why) | Rules of Hooks | Top level only, because React tracks Hooks by call order |
| [3.11](#311-stale-closures) | Stale closures | Functional updates, correct dependencies, refs, `useEffectEvent` |
| [4](#4-class-components-legacy) | Classes | Lifecycles map to Effects; error boundaries still need classes |
| [5](#5-component-patterns) | Patterns | Custom Hooks first; compound components for flexible UI; Portals escape clipping |
| [6](#6-state-management) | State | URL, server (React Query), form, local, global: pick by type of state |
| [6.2](#62-context-api--its-limits) | Context | Passes values; all consumers re-render on change |
| [7](#7-performance-optimization) | Performance | Measure → simplest fix → measure again; structure before memoisation |
| [7.4](#74-code-splitting-reactlazy-suspense-dynamic-import) | Code splitting | `React.lazy` + Suspense, by route first |
| [7.5](#75-list-virtualization-windowing) | Long lists | Virtualise |
| [7.9](#79-web-vitals-lcp-inp-cls) | Web Vitals | LCP ≤ 2.5 s, INP ≤ 200 ms, CLS ≤ 0.1 |
| [7.11](#711-react-compiler) | React Compiler | Automatic memoisation for code following the Rules of React |
| [8](#8-concurrent-react-react-18) | Concurrent React | Interruptible rendering; `useTransition` for your setter, `useDeferredValue` for a value |
| [9](#9-react-19) | React 19+ | Actions, `useOptimistic`, `use()`, `ref` as prop, `<Activity>`, `<ViewTransition>` |
| [10.1](#101-csr-vs-ssr-vs-ssg-vs-isr) | Rendering | CSR, SSR, SSG, ISR: decide per page |
| [10.2](#102-react-server-components-rsc-vs-client-components) | RSC | Server-only code that ships no JS; `'use client'` marks the boundary |
| [10.3](#103-hydration--hydration-mismatches) | Hydration | First client render must match the server HTML |
| [10.6](#106-nextjs-15--16-cache-components-proxyts--async-apis) | Next.js 16 | Explicit caching with `"use cache"`; `updateTag` for read-your-writes; `await params` |
| [11](#11-routing) | Routing | History API; server must fall back to `index.html`; shareable state goes in the URL |
| [12](#12-forms) | Forms | React Hook Form + Zod; validate on the server too; guard against double submit |
| [13](#13-error-handling--resilience) | Errors | Boundaries catch render errors only; use several; report with source maps |
| [14](#14-testing) | Testing | Test behaviour; integration focus; `getBy`/`queryBy`/`findBy`; MSW for the network |
| [15](#15-styling) | Styling | CSS Modules or Tailwind by default; avoid runtime CSS-in-JS for new work |
| [16](#16-typescript-with-react) | TypeScript | `ReactNode` for children; discriminated unions remove impossible states |
| [17](#17-security) | Security | React escapes output; server enforces access; no secrets in the bundle |
| [18](#18-accessibility-a11y) | Accessibility | Semantic HTML first; manage focus; keyboard support; test with a screen reader |
| [19](#19-architecture-senior--staff) | Architecture | Organise by feature; micro-frontends only for org scale; incremental migrations |
| [20](#20-javascript-fundamentals) | JavaScript | Closures, `this` by call site, microtasks before macrotasks, `let` per iteration |
| [21](#21-machine-coding--live-coding) | Machine coding | Clarify, plan state, make it work, then edge cases; talk as you go |
| [22](#22-behavioural--experience) | Behavioural | STAR with real numbers; say "I"; show trade-offs |
| [23](#23-html--css-fundamentals) | HTML & CSS | `border-box`; specificity columns; stacking contexts; Flexbox 1-D, Grid 2-D |
| [24](#24-browser-internals) | Browser | DOM + CSSOM → layout → paint → composite; animate `transform`/`opacity`; capture and bubble |
| [25](#25-network--data-fetching-fundamentals) | Network | CORS is browser-enforced, server-configured; hashed assets + `no-cache` HTML; `fetch` doesn't reject on 404 |
| [26](#26-memory-leaks) | Memory leaks | Every setup needs a cleanup; find with heap snapshots and Retainers |
| [27](#27-web-workers-service-workers--pwa) | Workers | Web Worker = thread; Service Worker = network proxy; PWA = installable + offline |
| [28](#28-practical-features) | Features | Presigned, resumable uploads; `AnimatePresence` for exits; dnd-kit for accessible drag |
| [29](#29-frontend-system-design-case-studies) | System design | RADIO; normalised data; cursor pagination; optimistic UI; plan for failure |

---

> **Final tip:** For every topic above, practise saying the **one-paragraph explanation out loud**, then the **scenario answer**, then a **real example from your own work**. Interviewers remember candidates who connect concepts to real production experience.
