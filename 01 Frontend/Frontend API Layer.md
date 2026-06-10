---
tags: [frontend, reference]
repo: achieveup-frontend
file: src/services/api.ts
---

# Frontend API Layer (`src/services/api.ts`)

**Every** HTTP request the web app makes lives in this one 315-line file. Understanding it means you know the entire frontend↔backend contract.

## The Axios setup (top of file)
```ts
const API_BASE_URL = process.env.REACT_APP_API_URL || 'http://localhost:5001';
const api = axios.create({ baseURL: API_BASE_URL, ... });
```
Two **interceptors** do the invisible work:
- **Request interceptor:** reads the [[JWT Authentication|JWT]] from `localStorage` and adds `Authorization: Bearer <token>` to every call.
- **Response interceptor:** on a `401` response, wipes the token and hard-redirects to `/login`. (Network errors deliberately don't log you out.)

## The API groups
Each exported object groups related endpoints. Frontend components import only the groups they need.

| Export | Backend prefix | Used for |
|---|---|---|
| `authAPI` | `/auth/*` | login, signup, verify, me, profile, password, validate-canvas-token |
| `skillMatrixAPI` | `/achieveup/matrix/*`, `/achieveup/ai/suggest-skills` | CRUD skill matrices, AI skill suggestions, import matrices from another course |
| `courseDescriptionAPI` | `/achieveup/course-description/*` | Get/update course description (extra context for the AI) |
| `skillAssignmentAPI` | `/achieveup/skills/*`, `/achieveup/ai/*` | Assign skills to questions, AI analyze/bulk-assign, fetch existing assignments |
| `badgeAPI` | `/achieveup/badges/*` | Generate badges, fetch a student's earned badges, **public** badge endpoint (no auth) |
| `progressAPI` | `/achieveup/progress/*` | Get/update a student's skill progress |
| `analyticsAPI` | `/achieveup/graphs/*`, `/achieveup/export|import` | Individual graphs, course data export/import |
| `canvasAPI` | `/canvas/*` | Courses/quizzes/questions via the user's Canvas token; `test-connection` |
| `canvasInstructorAPI` | `/canvas/instructor/*` | Instructor-scoped versions of the above + enrollment/submissions |
| `questionAnalysisAPI` | `/achieveup/questions/*` | Per-question AI analysis & suggestions |
| `instructorAPI` | `/achieveup/instructor/*` | Dashboard stats, course student-analytics (Progress page), **force-sync** |

## ⚠️ Things to watch
- **~10 functions in this file are dead code** (verified June 2026): they point at endpoints that don't exist in the backend (`instructorAPI.suggestSkillsForCourse`, `.analyzeQuestionsWithAI`, `.bulkAssignSkillsWithAI`, `.assessStudentSkills`, `.generateWebLinkedBadges`, `badgeAPI.getCourseBadges`, `canvasInstructorAPI.getQuizSubmissions`, `.getCourseEnrollment`, `.validateInstructorToken`, `.getInstructorQuestions`) **and no component calls them**. Every endpoint the live screens use was verified to exist. Don't wire new UI to these without adding the backend route first. Logged in [[Open Questions]].
- Naming trap: `SkillAssignmentInterface.tsx` defines its own local `analyzeQuestionsWithAI()` which calls the **working** `skillAssignmentAPI.analyzeQuestions` (`/achieveup/ai/analyze-questions`) — not the dead `instructorAPI` function of the same name.
- `skillMatrixAPI.get(courseId)` vs `getAllByCourse(courseId)` — the latter exists because one course can have *multiple* matrices.
- The Progress page's **Sync Now** button calls `instructorAPI.forceSyncCourse()`; a `202` response means "running in background" and the page then polls every 15s. See [[Flow - Background Canvas Sync]].

Related: [[API Route Reference]] (backend side of this contract) · [[Frontend Auth Flow]]
