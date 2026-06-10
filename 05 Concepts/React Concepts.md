---
tags: [concept, frontend]
---

# React Concepts (taught through this codebase)

Everything React-y this project uses, with pointers to real examples in `achieveup-frontend`. Read alongside [[Frontend Overview]].

## The one-sentence mental model
React = your UI is a **function of state**: you write components (functions returning JSX), React re-runs them whenever their state changes and updates the page to match.

## Components & JSX
A component is a function returning HTML-ish markup (JSX). Capitalized = component.
```tsx
const Button: React.FC<ButtonProps> = ({ children, onClick }) => (
  <button onClick={onClick} className="...">{children}</button>
);
```
📍 Simplest real examples: `src/components/common/Button.tsx`, `Card.tsx`, `Input.tsx`. Components compose: `App` → `Layout` → `Navigation` + page content.

## Props vs State
- **Props** = inputs passed from parent (`<BadgesDashboard courseId={selectedCourse} />` in `App.tsx` — `courseId` is a prop).
- **State** = a component's own memory, via the `useState` hook:
```tsx
const [courses, setCourses] = useState<any[]>([]);   // App.tsx, StudentProgress
const [loading, setLoading] = useState(true);
```
Calling `setCourses(...)` triggers a re-render. You never mutate state directly.

## Hooks you'll see here
| Hook | What it does | Real usage |
|---|---|---|
| `useState` | Local state | everywhere — data, loading flags, modals (`showModal` in `App.tsx`) |
| `useEffect` | Run side effects (data fetching) after render; deps array controls when | `useEffect(() => { loadCourses(); }, [])` = "on mount"; `useEffect(..., [selectedCourse])` = "when course changes" — both in `StudentProgress` |
| `useContext` | Read shared state without prop-drilling | `useAuth()` wraps it — see below |

The standard **fetch pattern** used on every screen:
```tsx
const [data, setData] = useState(null);
const [loading, setLoading] = useState(true);
const [error, setError] = useState('');
useEffect(() => {
  api.getThing()
    .then(res => setData(res.data))
    .catch(() => setError('...'))
    .finally(() => setLoading(false));
}, []);
// render: if (loading) spinner; if (error) message; else data
```

## Context (app-wide state)
`createContext` + a `<Provider>` lets any descendant read shared values. This app has exactly one: **AuthContext**.
- `AuthProvider` wraps the whole app in `App.tsx`.
- Any component calls `const { user, isAuthenticated, logout } = useAuth();`
📍 `src/contexts/AuthContext.tsx` — also a great example of a **custom hook** (`useAuth`) that errors if used outside its provider. Details: [[Frontend Auth Flow]].

## Routing (react-router-dom)
The URL decides which component renders — no page reloads:
```tsx
<Routes>
  <Route path="/login" element={<Login />} />
  <Route path="/badges/:studentId" element={<StudentPublicBadges />} />  // :param
</Routes>
```
📍 All in `App.tsx`. Note the **wrapper-component pattern**: `ProtectedRoute` checks auth and either renders `children` or `<Navigate to="/login" />` — composition instead of middleware.

## Conditional rendering & lists
```tsx
{loading && <Spinner />}                       // render-if
{error ? <ErrorBox /> : <Table />}             // either/or
{students.map(s => <tr key={s.id}>...</tr>)}   // lists need a stable `key`
```
📍 `StudentProgress` in `App.tsx` is a master class in all three.

## How this app starts (the chain)
`public/index.html` (empty `<div id="root">`) → `src/index.tsx` (`createRoot(...).render(<App/>)`) → `App.tsx` (providers + router) → page components → [[Frontend API Layer|api.ts]] for data.

## Vocabulary cheat sheet
| Term | Meaning |
|---|---|
| Render / re-render | React re-running your component function after state/props change |
| Mount / unmount | Component appearing in / leaving the page |
| SPA | Single-page app — one HTML file, JS swaps views (why Netlify needs the redirect rule in `netlify.toml`) |
| Controlled input | Form field whose value lives in React state (`value={x} onChange={...}`) |
| CRA | Create React App — the build tooling (`react-scripts`) this repo uses |

Related: [[TypeScript Notes]] · [[Tailwind CSS]]
