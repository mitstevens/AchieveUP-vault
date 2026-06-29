---
tags: [team, issues, code-quality, reference]
created: 2026-06-29
---

# Issue Tracker

A single, ranked, **verified** list of every issue found across the personal notes and the AI-generated vault docs. Sources: [[Frontend Code Quality Ratings]], [[Open Questions]], the `Personal Notes/` folder, the file walkthroughs, and [[Frontend Auth Flow]].

> [!info] How to read this
> - **Priority** is out of 10. **10 = fix immediately**, **1 = skip unless bored.**
> - **✓** = confirmed in source on 2026-06-29 (line numbers checked, not copied).
> - **✓\*** = confirmed by the vault's own code-verification; category spot-checked.
> - **⚠** = real in code, but exact impact depends on Heroku/prod config we can't see locally.
> - Line numbers are from the repos as of 2026-06-29 and will drift as code changes.

> [!success] Verification note
> Every high-impact item was re-checked directly against the actual source. Nothing checked was a phantom — the existing vault notes held up. The only caveats are the ⚠ items, whose severity depends on deployment state (ask whoever owns Heroku).

---

## 🔴 Critical — security / runtime (8–10)

| Pri | Issue | Where | Status |
|---|---|---|:--:|
| **10** | **JWT secret has a public hardcoded fallback** (`"achieveup-secret-key-change-in-production"`). If the env var is unset in prod, *anyone can forge a valid token for any user/role.* **Fix:** remove the usable default and fail-fast on startup if missing. **Also:** verify Heroku has it set. | `config.py:61`, `achieveup_auth_service.py:23` | ✓ / ⚠ |
| **8** | **Login & Signup ignore the boolean they get back.** A blocked/failed `login()` returns `false` without throwing, so the app still `navigate('/')`s and toasts "Welcome back, instructor!" — fake success. | `Login.tsx:28-30`, `Signup.tsx:43-44` | ✓ |
| **8** | **`generate_jwt_token` calls `asyncio.run()` inside the running event loop** → `/auth/refresh-token` throws at runtime. Currently unused by the frontend, so latent — but broken. | `achieveup_auth_service.py:66` | ✓ |

## 🟠 High — functional bugs & misleading data (6–7)

| Pri | Issue | Where | Status |
|---|---|---|:--:|
| **7** | **SPA-breaking navigation:** raw `<a href>` (and `window.location.href` in the nav help modal) instead of `<Link>` → full page reload, drops React/in-memory state. | `App.tsx:290,525,531`; `Dashboard.tsx:494`; `Navigation.tsx` | ✓ |
| **7** | **Dashboard fabricates "Total Students"** from a hash of the course name when the real count is 0, shown as a real metric — and the ~30-line block is duplicated in the catch handler. | `Dashboard.tsx:177-186` (+dup ~216-242) | ✓ |
| **7** | **Canvas host split-brain:** the sync loop hardcodes `canvas.instructure.com` while the rest uses `CANVAS_API_URL`. If Heroku points at UCF Webcourses, the two halves of one sync hit different Canvas instances → UCF data may never sync. See [[Open Questions]]. | `app.py:174`, `config.py:62` | ✓ / ⚠ |
| **7** | **Backend role checks are cosmetic on the client** — real enforcement must live in the routes. State this as a rule before building the student portal. | (principle) | — |
| **6** | **Signup hardcodes `role/canvasTokenType='instructor'`** → no student can register. This is the student-portal blocker. | `AuthContext.tsx`, `Signup.tsx:40` | ✓ |
| **6** | **`User.canvasTokenType` typed `'instructor'` only** → every `'student'` branch in the app is dead code; blocks typing the portal. | `types/index.ts` | ✓ |
| **6** | **Backend route collision:** `/instructor/analyze-questions-with-ai` & `/instructor/bulk-assign-skills-with-ai` are registered in **two** blueprints — one silently shadows the other. | `instructor_routes.py:485,545` + `achieveup_routes.py:1485,1520` | ✓ |
| **6** | **SkillAssignment fabricates fake skill matrices** on 404/403/500 → instructor believes matrices exist. | `SkillAssignmentInterface.tsx:578` (used 4×) | ✓ |

## 🟡 Medium — dead code, structure, secondary fake data (4–5)

| Pri | Issue | Where | Status |
|---|---|---|:--:|
| **5** | **`StudentProgress` (~600 lines) + 130-line modal declared inline in `App.tsx`** → extract to `pages/`. Directly eases the portal split. | `App.tsx` | ✓ |
| **5** | **`instructor_routes.py` (17 `/instructor/*` routes) has zero callers** — dead duplicate of `/achieveup/instructor/*`; deleting also kills the collision above. | backend | ✓ |
| **5** | **`/badges-test` debug harness routed in the live app** → any logged-in user can look up *any* student's badges by raw ID (mild data exposure). | `StudentBadgesTest.tsx` | ✓ |
| **5** | **JWT in `localStorage`** → XSS token-theft risk vs httpOnly cookie. Bigger architectural change — team decision. See [[Frontend Auth Flow]]. | `AuthContext.tsx` | ✓ |
| **5** | **Three different password policies:** Signup (8 + complexity) / Login (8) / Settings (6). | three files | ✓ |
| **5** | `SkillMatrixCreator` silently falls back to ~80 lines of hardcoded mock skill suggestions. | `SkillMatrixCreator.tsx` | ✓ |
| **5** | `BadgesDashboard` sets `earnedAt` to a **random** past date → "Earned on …" is fabricated. | `BadgesDashboard.tsx:123` | ✓ |
| **4** | **Debug/console debris (~80 logs)**, incl. AuthContext logging the **login email** and `Input.tsx` logging email/password props every render. | `AuthContext.tsx`, `Input.tsx:18`, … | ✓ |
| **4** | **"🔍 Test Authentication" debug button** shipped in prod UI. | `SkillAssignmentInterface.tsx:992` | ✓ |
| **4** | **User-facing toast leaks dev talk:** "Backend Issue… the backend team needs to implement this endpoint." | `SkillMatrixCreator.tsx:755` | ✓ |
| **4** | **Double `<Toaster>`** mounted → toasts can render twice. | `index.tsx:13` + `App.tsx:738` | ✓ |
| **4** | Sync-Now `setInterval` not cleared on unmount → `setState` on unmounted component + wasted polls. | `App.tsx:232` | ✓ |
| **4** | **`AnalyticsDashboard.tsx` (560 lines) is dead** — never imported/routed. | frontend | ✓ |
| **4** | `BadgesDashboard` has an **empty `try {}` + `catch`** → catch unreachable, its fallback never runs. | `BadgesDashboard.tsx:74-91` | ✓ |
| **4** | **Dashboard nav `href: '/'` never matches `/dashboard`** → Dashboard tab never shows the active state. | `Navigation.tsx:19` | ✓ |
| **4** | ~10 dead `api.ts` functions point at endpoints that don't exist (trap for new UI). See [[Frontend API Layer]]. | `api.ts` | ✓* |
| **4** | `generateMockActivity` uses arbitrary "x min ago" timestamps. | `Dashboard.tsx` | ✓ |
| **4** | **README says backend is Node/Express; it's Python Quart.** Misleads new teammates. | frontend `README.md` | ✓* |

## 🟢 Low — cleanup & style (1–3)

| Pri | Issue | Status |
|---|---|:--:|
| **3** | `isInstructor` role-check duplicated 3× in AuthContext; `getImportStatus` duplicated (`api.ts:83,111`); `QuestionAnalysis`/`QuestionSuggestion` byte-identical interfaces. | ✓ |
| **3** | `{false && (…)}` dead JSX block + duplicated course-code helpers in SkillAssignment. | ✓ |
| **3** | Open team/deploy unknowns: canonical repo fork? live prod URL? dyno always-on? real FERPA data in prod DB? which `config.py` collections are used? See [[Open Questions]]. | ✓* |
| **2** | 17 `AsyncIOMotorClient` instances (incl. a per-call one in `generate_jwt_token`) → share one client. | ✓ |
| **2** | Hardcoded stat cards (not array-driven), `key={index}`, deprecated `onKeyPress`, mixed 2-/4-space indentation. | ✓ |
| **2** | `Button` `warning`/`primary` variants visually identical; Skill Matrix & Assignment share an icon. | ✓ |
| **2** | Extension repo has unrelated root-level "YouTube demo" files (`manifest.json`/`index.html`/`popup.js`) — confirm safe to delete. | ✓* |
| **1** | Move static `instructorQuickActions` outside the component (minor perf). | ✓ |

---

## 🎯 Recommended first sprint

Tackle what's **either dangerous or that we'll have to touch anyway** building the student portal — don't boil the ocean.

1. **`ACHIEVEUP_JWT_SECRET`** — confirm it's set on Heroku **and** make the code refuse to boot without it. *(Pri 10, ~30 min.)*
2. **Login/Signup ignored-boolean** — handle the return so failed logins don't fake-succeed. *(Pri 8 — and we're reworking login redirect anyway.)*
3. **Un-hardcode signup + widen the `User` type** — the portal can't exist without this. *(Pri 6.)*
4. **Backend route collision + delete dead `instructor_routes.py`** — clean the routing surface *before* adding student endpoints. *(Pri 6/5.)*
5. **Kill the fabricated Dashboard student count** (show "—") — we're rebuilding dashboards for the portal regardless. *(Pri 7.)*
6. **Cheap-but-embarrassing debris:** remove the "Test Authentication" button, the "backend team" toast, the email `console.log`, the double `<Toaster>`, and gate/remove `/badges-test`. *(~half a day, big polish-per-effort.)*

> [!note] Defer
> `localStorage`→cookie migration (bigger — isolate it), deleting `AnalyticsDashboard`, the full `<a>`→`<Link>` sweep (do opportunistically), and all pure-style items.

> [!warning] Investigate before fixing
> The **Canvas-host split-brain** (Pri 7) is investigate-first: answer *"what is `CANVAS_API_URL` set to on Heroku?"* before writing any fix.

---

## Related notes
- [[Frontend Code Quality Ratings]] — per-file scores and detail
- [[Open Questions]] — deployment/team unknowns
- [[Frontend Auth Flow]] / [[Flow - Authentication]] — auth specifics
- [[Frontend API Layer]] — dead endpoint functions
