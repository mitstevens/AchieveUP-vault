---
tags: [frontend, overview]
repo: achieveup-frontend
---

# Frontend Overview — `achieveup-frontend`

The **AchieveUp instructor portal**: a single-page React app where instructors create skill matrices, map quiz questions to skills, and monitor student mastery, risk, and badges.

- **Stack:** React 18 + TypeScript, [[Tailwind CSS]] (UCF gold theme), React Router, Axios, react-hot-toast, react-hook-form, lucide-react icons.
- **Created with:** Create React App (`react-scripts`) — so there's no visible webpack config; `npm start` runs a dev server on port 3000.
- **Deploys:** Netlify (`netlify.toml`), production at achieveup.netlify.app.
- **Backend URL:** `REACT_APP_API_URL` env var, defaults to `http://localhost:5001`.

## Mental model (read this before the code)
```mermaid
flowchart LR
    index["index.tsx<br/>(entry)"] --> App["App.tsx<br/>(routes)"]
    App --> Auth["AuthContext.tsx<br/>(who is logged in?)"]
    App --> Pages["pages/ & components/<br/>(screens)"]
    Pages --> API["services/api.ts<br/>(ALL backend calls)"]
    API -->|"Axios + JWT header"| BE["knowgap-backend"]
```

Three structural rules that make this codebase easy to navigate:
1. **Every backend call goes through `src/services/api.ts`** — grouped into exported objects (`authAPI`, `skillMatrixAPI`, `badgeAPI`, `instructorAPI`, …). If you wonder "how does screen X get its data," find the API group it imports. See [[Frontend API Layer]].
2. **Auth state lives in one place** — `src/contexts/AuthContext.tsx`, a React Context wrapping the whole app. See [[Frontend Auth Flow]] and [[React Concepts#Context]].
3. **Routes = screens** — `App.tsx` declares every URL. Each route is wrapped in `ProtectedRoute` (redirects to `/login` if unauthenticated) and `Layout` (nav bar shell).

## The screens (routes in `App.tsx`)
| Route | Component | What the instructor does there |
|---|---|---|
| `/login`, `/signup` | `pages/Login`, `pages/Signup` | Email/password auth; optional Canvas token at signup |
| `/dashboard` | `pages/Dashboard` | Course list, workflow progress, recent activity |
| `/skill-matrix` | `components/SkillMatrixCreator` | Step 1: define course skills (AI suggestions available) |
| `/skill-assignment` | `components/SkillAssignmentInterface` | Step 2: map quiz questions → skills (AI bulk-assign) |
| `/progress` | `StudentProgress` (defined *inline in App.tsx*) | Step 3: per-student mastery table, risk levels, Sync Now button |
| `/settings` | `pages/Settings` | Canvas token management, profile |
| `/badges/:studentId` | `pages/StudentPublicBadges` | **Public** (no login) shareable badge page |
| `/badges-test` | `components/StudentBadgesTest` | Dev/test harness for badge display |

⚠️ Quirk: `StudentProgress` (~600 lines) is defined inside `App.tsx` rather than its own file — likely a candidate for refactoring.

## Where to go next
- [[Frontend File Guide]] — every file explained
- [[Flow - Instructor Skill Workflow]] — the workflow these screens implement
- [[React Concepts]] — the React features this app uses, with examples from this code
