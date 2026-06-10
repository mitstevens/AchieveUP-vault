---
tags: [frontend, reference]
repo: achieveup-frontend
---

# Frontend File Guide

Every file in `achieveup-frontend`, what it does, and what it links to. Paths relative to repo root.

## Root config files
| File | Purpose |
|---|---|
| `package.json` | Dependencies & scripts. `npm start` (dev, :3000), `npm test`, `npm run build`. CRA-based (`react-scripts`). |
| `tsconfig.json` | TypeScript compiler settings (strict mode, JSX). See [[TypeScript Notes]]. |
| `tailwind.config.js` | [[Tailwind CSS]] theme — defines the `ucf-gold` color etc. |
| `postcss.config.js` | Pipes Tailwind into CRA's CSS build. You will likely never touch it. |
| `netlify.toml` | Netlify deploy config (build command, SPA redirect so React Router URLs work on refresh). |
| `public/` | Static HTML shell (`index.html` has the `<div id="root">` React mounts into), favicon, manifest. |

## `src/` — entry & core
| File | Purpose |
|---|---|
| `index.tsx` | **Entry point.** Renders `<App />` into the DOM. Imports `index.css`. |
| `index.css` | Global styles + Tailwind `@tailwind` directives. |
| `App.tsx` | **Heart of the app.** Declares all routes, the `ProtectedRoute` wrapper, and (inline) the entire `StudentProgress` page incl. the student-detail modal and the Sync Now polling logic. |
| `types/index.ts` | **All shared TypeScript interfaces** (`User`, `SkillMatrix`, `Badge`, `CanvasCourse`, request/response shapes). Read this to learn the data model the frontend expects. |

## `src/contexts/`
| File | Purpose |
|---|---|
| `AuthContext.tsx` | React Context providing `{ user, isAuthenticated, loading, login(), signup(), logout(), ... }` to the tree. Persists JWT in `localStorage`, verifies it on app load via `/auth/verify`. See [[Frontend Auth Flow]]. |

## `src/services/`
| File | Purpose |
|---|---|
| `api.ts` | **The only place HTTP happens.** Axios instance + interceptors (attach JWT, redirect to `/login` on 401) + ~11 grouped API objects. Full breakdown: [[Frontend API Layer]]. |
| `api.test.ts` | Unit tests for the API layer. |

## `src/pages/` — top-level screens
| File | Purpose |
|---|---|
| `Login.tsx` / `Signup.tsx` | Auth forms (react-hook-form). Signup optionally collects a Canvas API token. |
| `Dashboard.tsx` | Instructor landing page: courses from Canvas, workflow status, quick links. |
| `Settings.tsx` | Manage Canvas API token (validate/test connection), profile, password. |
| `StudentPublicBadges.tsx` | Public badge showcase at `/badges/:studentId` — uses the unauthenticated `/achieveup/public/badges/...` endpoint so students can share a link. |
| `*.test.tsx` | Component tests (React Testing Library). |

## `src/components/`
| File | Purpose |
|---|---|
| `Layout/Layout.tsx` | Page shell: renders `Navigation` + the page content. Wraps every protected route. |
| `Layout/Navigation.tsx` | Top nav bar — links to Dashboard / Skill Matrix / Assignment / Progress / Settings, logout button. |
| `common/Button.tsx`, `common/Input.tsx`, `common/Card.tsx` | Reusable styled primitives used across forms/pages. Good first files to read to learn the component style. |
| `SkillMatrixCreator/SkillMatrixCreator.tsx` | **Workflow step 1.** Create/edit skill matrices for a course; calls `skillMatrixAPI` incl. AI `suggest-skills`; supports importing matrices from another course. |
| `SkillAssignmentInterface/SkillAssignmentInterface.tsx` | **Workflow step 2.** Pick course → quiz → questions; assign skills per question; AI analyze / bulk-assign via `skillAssignmentAPI`. |
| `BadgesDashboard/BadgesDashboard.tsx` | Course-level badge overview, embedded at the bottom of the Progress page. |
| `AnalyticsDashboard/AnalyticsDashboard.tsx` | Charts/analytics view (uses `analyticsAPI` / `instructorAPI`). |
| `StudentBadgesTest/StudentBadgesTest.tsx` | Internal test page for badge rendering. |

## `src/integration/` & `src/test-utils/`
| File | Purpose |
|---|---|
| `integration/App.integration.test.tsx` | App-level integration tests (routing + auth together). |
| `test-utils/test-runner.ts` | Shared test helpers. |

> [!note] Pattern to internalize
> Screen component → imports an API group from `api.ts` → `useEffect` fetches on mount → `useState` holds data/loading/error → render. Once you've read one screen (try `SkillMatrixCreator`), you've effectively read them all. See [[React Concepts]].
