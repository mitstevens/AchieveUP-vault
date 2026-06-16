---
tags: [frontend, reference, code-quality]
repo: achieveup-frontend
---

# Frontend Code Quality Ratings

A per-file quality rating of every core file in `achieveup-frontend/src`. Scores judge **best practices, bugs, and overall design/efficiency** — not whether the feature works. Test files and trivial config are skipped. Paths relative to repo root.

Every bug and inconsistency below was verified against the actual source. See [[Frontend File Guide]] for what each file *does*.

> [!summary] Overall: ~6.5/10
> The app is functional and the visual/UX layer is genuinely strong. What drags it down is consistent across files: **fabricated mock data presented as real**, **leftover debug code (console.logs, test buttons, dev-only toast messages)**, **heavy copy-paste duplication**, and a few **dead-code blocks**. None of it is catastrophic; almost all of it is cleanup.

---

## Entry & core

| File | Score | Notes |
|---|---|---|
| `index.tsx` | **7/10** | Clean, standard CRA entry. **One real issue:** it renders `<Toaster position="top-right" />` *and* `App.tsx` renders a second `<Toaster>` (App.tsx:738). Two toasters are mounted, so every toast can render twice. Remove one (keep the configured one in `App.tsx`). |
| `App.tsx` | **5/10** | Does too much. The entire `StudentProgress` page (~600 lines incl. modal + Sync polling) is declared inline here instead of in `pages/`. The Sync-Now `setInterval` (line 232) is never cleared on unmount → timer leak if the user navigates away mid-poll. Duplicate `<Toaster>` (see above). Heavy `any` usage (`courses`, `selectedStudent`). Routing itself is clean and correct. |
| `types/index.ts` | **8/10** | Well-organized, genuinely useful as the data-model reference. Minor: `QuestionAnalysis` (line 279) and `QuestionSuggestion` (line 286) are byte-for-byte identical interfaces — collapse to one. `User.canvasTokenType` is hard-typed `'instructor'` only, which quietly makes all the "student" branches elsewhere dead code. |

## Context & services

| File | Score | Notes |
|---|---|---|
| `contexts/AuthContext.tsx` | **6/10** | Solid auth flow with sensible token handling and graceful `me()` fallback. **Duplication:** the `isInstructor` role-check block is copy-pasted 3× (lines 52-55, 106-109, 223-226) — extract a helper. Lots of `console.log` left in (6), including logging the email being logged in. Logic is correct otherwise. |
| `services/api.ts` | **7/10** | Clean, well-typed axios layer with good interceptors. `getImportStatus` is defined identically in both `skillMatrixAPI` (line 83) and `skillAssignmentAPI` (line 111). `canvasAPI` and `canvasInstructorAPI` overlap heavily (`getInstructorCourses`/`Quizzes`/`Questions` exist in both with subtly different signatures) — easy to call the wrong one. Otherwise the strongest non-trivial file. |

## Common components

| File | Score | Notes |
|---|---|---|
| `common/Button.tsx` | **8/10** | Clean, idiomatic, good use of `clsx` + variant/size maps. `warning` and `primary` variants are visually identical (both `bg-ucf-gold`) — probably unintended. Otherwise textbook. |
| `common/Card.tsx` | **8/10** | Simple, correct, reusable. No complaints worth raising. |
| `common/Input.tsx` | **5/10** | **Leftover debug code:** lines 16-26 `console.log` on every render when `name` is `email`/`password`/`matrixName` — fires real console spam on the Settings page. Also, because it's a plain function component it does **not** forward `ref`, so it can't be used with react-hook-form `register()`. It currently isn't (forms use raw `<input>` or controlled mode), so no live bug — but it's a trap. Remove the logging; wrap in `forwardRef`. |

## Layout

| File | Score | Notes |
|---|---|---|
| `Layout/Layout.tsx` | **9/10** | Tiny, correct, does exactly one thing. |
| `Layout/Navigation.tsx` | **6/10** | Good responsive nav + help modal. **Real bug:** the Dashboard item uses `href: '/'`, but the dashboard actually lives at `/dashboard` (`/` just redirects). So `location.pathname === item.href` is never true there and **Dashboard never shows the active state**. "Skill Matrix" and "Skill Assignment" share the same `Target` icon. Help-modal CTAs use `window.location.href` (full page reload) instead of router navigation. The huge static help modal could be its own component. |

## Pages

| File | Score | Notes |
|---|---|---|
| `pages/Login.tsx` | **6/10** | Clean form, good validation/UX. **Bug:** `onSubmit` ignores the boolean `login()` returns — a non-instructor login returns `false` *without throwing*, so the code still `navigate('/')`s and shows a "Welcome back!" toast on a failed login. Shares ~80% of its markup (email field, password+toggle, identical "What you can do" feature list) with `Signup.tsx`. |
| `pages/Signup.tsx` | **6/10** | Same strengths and the same ignored-boolean issue as Login. Password rules here (8 chars + upper/lower/number) are stricter than Login's (8 chars) and Settings' (6 chars) — **three different password policies** in one app. Markup heavily duplicated with Login — extract a shared `AuthField`/feature-list. |
| `pages/Settings.tsx` | **6/10** | Functional and reasonably organized. **Duplication:** the entire token-type radio + token `<Input>` block is copy-pasted between the "no token set" branch (388-429) and the "editing" branch (432-474). Uses the controlled `Input` (so it triggers Input.tsx's debug log). Password min length (6) contradicts Signup (8). Good error handling on the API calls. |
| `pages/Dashboard.tsx` | **4/10** | Looks great, but the data is partly **fabricated**. When the backend returns 0 students, it invents a "Total Students" number from a hash of the course name (lines 171-199) — and that *exact ~30-line block is duplicated verbatim* in the catch handler (216-242). Presenting made-up counts as real metrics is a design problem, not just a code one. The "student" branch is dead (app is instructor-only). Extract the mock logic, or better, show "—" instead of inventing data. |

## Workflow components

| File | Score | Notes |
|---|---|---|
| `SkillMatrixCreator/SkillMatrixCreator.tsx` | **5/10** | Big (1.4k lines) but the feature is genuinely complex, so size is somewhat earned. Problems: the unique-matrix-name `do…while` loop is duplicated **3×** (296-311, 698-712, 724-740); a user-facing error toast literally tells the end user *"Backend Issue: …the backend team needs to implement this endpoint"* (753-757) — dev messaging leaking to users; inconsistent indentation (the helper block 67-133 is mis-indented vs the rest of the component); 13 `console.log`s; ~80 lines of hardcoded mock skill suggestions used as a silent fallback. Deprecated `onKeyPress`. |
| `SkillAssignmentInterface/SkillAssignmentInterface.tsx` | **5/10** | The most feature-rich screen, and it shows — but it carries the most debris. **Debug UI left in production:** a "🔍 Test Authentication" button + `validateAndTestToken()` (917, 990-996). A permanently-dead `{false && (…)}` JSX block (1369). **Two** Save buttons (in-form submit at 1505 + sticky button at 1534) with mismatched disabled rules. Fabricates entire fake skill matrices on 404/403/500 (`generateMockMatrices`), which can mislead an instructor into thinking matrices exist. `getSection`/`getBaseCourseCode`/`findPastCourse` are duplicated from `SkillMatrixCreator`. 15 `console.log`s. |
| `BadgesDashboard/BadgesDashboard.tsx` | **5/10** | UI and expand/collapse logic are nice. **Real dead code:** lines 74-91 are an *empty* `try {}` followed by a `catch` — the catch can never run, so its "empty badges" fallback is unreachable. **Fabricated data:** `earnedAt` is set to a *random* past date (line 123) rather than a real earned timestamp. 80%-threshold badge logic is reasonable. Uses 4-space indent while most files use 2 (inconsistent across the codebase). |
| `AnalyticsDashboard/AnalyticsDashboard.tsx` | **4/10** | Competent recharts dashboard with good empty-state handling — **but it is dead code: nothing imports or routes it** (only its props type is referenced). 560 lines that never render. The Sync-Now button + course selector is duplicated within the file (no-data view vs main view) and a third time in `App.tsx`. If you want it, wire it to a route; otherwise delete it. |

## Public / test pages

| File | Score | Notes |
|---|---|---|
| `pages/StudentPublicBadges.tsx` | **8/10** | The cleanest substantial file. Good use of query-param + API fallback for the student name, sensible `document.title` effect, tidy loading/error/empty states, nice card design. Minor: `getBadgeColor` would throw if `badge_level` were ever null. Use this as the style template for the others. |
| `StudentBadgesTest/StudentBadgesTest.tsx` | **6/10** | Clean and readable, but it's an **internal debug harness routed into the live app** at `/badges-test` (search any student's badges by raw ID). Fine as a tool, but it shouldn't ship behind normal auth. Its `getBadgeLevelColor`/`getBadgeIconColor` helpers duplicate badge-color logic found in other badge components. |

---

## Cross-cutting themes (the stuff that repeats everywhere)

> [!warning] These are the patterns to fix once, globally — they'll lift several scores at once.
> 1. **Fabricated mock data presented as real** — student counts ([[Dashboard.tsx Walkthrough|Dashboard]]), skill suggestions (SkillMatrixCreator), skill matrices (SkillAssignmentInterface), badge earned-dates (BadgesDashboard). Either label it clearly as demo data or show an honest empty state.
> 2. **Leftover debug code** — ~80 `console.log`s across the app, a "Test Authentication" button, an `Input` that logs every render, and user-facing toasts that talk about "the backend team."
> 3. **Copy-paste duplication** — unique-name loop (3×), role check (3×), Sync-Now logic (3×), course-code helpers (2 files), auth-form markup (Login/Signup), token form (Settings).
> 4. **Dead code** — unused `AnalyticsDashboard` (560 lines), unreachable `try/catch` in BadgesDashboard, `{false && …}` block, dead "student" role branches.
> 5. **Inconsistencies** — 2- vs 4-space indentation, three different password policies, navigation done via `<Link>` *and* `<a href>` *and* `window.location.href` (the latter two force full reloads, defeating the SPA), deprecated `onKeyPress`.

### What's genuinely good
- The `common/` primitives, `Layout`, `api.ts`, `types`, and `StudentPublicBadges` are clean and idiomatic.
- Tailwind usage is consistent and the visual design is strong.
- Error handling is *defensively* thorough (almost too much) — the app rarely hard-crashes.
- The overall "screen → API group → useEffect → useState" pattern is consistent and easy to follow.
