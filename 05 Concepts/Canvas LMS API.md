---
tags: [concept, integration]
---

# Canvas LMS API

Canvas (by Instructure) is the LMS; UCF's instance is **webcourses.ucf.edu**. Its REST API (`/api/v1/...`) is the source of all course/quiz/student data in this project.

## Auth model used here
Personal **access tokens**: a user generates one in Canvas → Account → Settings → "New Access Token". Sent as `Authorization: Bearer <token>`. The token acts **as that user** — an instructor token can see enrollments/submissions; a student token only their own data. That's why the backend detects "token type."

> Tokens are why security matters so much in this codebase: they're stored Fernet-encrypted ([[Flow - Authentication]]), and the long-term TODO is migrating to OAuth2.

## Endpoints this project relies on (Canvas side)
| Canvas endpoint | Used for |
|---|---|
| `GET /api/v1/users/self` | Token validation (extension popup does this directly) |
| `GET /api/v1/courses` | Course lists (filtered by enrollment type for instructors) |
| `GET /api/v1/courses/:id/quizzes` | Quizzes per course |
| `GET /api/v1/courses/:id/quizzes/:qid/questions` | Quiz questions (for skill mapping & video topics) |
| `GET .../quizzes/:qid/submissions` + submission events/answers | Who answered what, correct/incorrect — feeds mastery & risk |
| `GET /api/v1/courses/:id/enrollments` | Student rosters |

Backend wrappers: `services/achieveup_canvas_service.py` (AchieveUp), `services/canvas_submissions_service.py` (submissions), `routes/course_utils.py` + `utils/course_utils.py` (legacy).

## Practical notes
- **Two hosts** appear in code: `canvas.instructure.com` (generic default) and `webcourses.ucf.edu` (UCF). The legacy `Tokens` docs store the host per user (`link` field); AchieveUp users currently default to `canvas.instructure.com` in the sync loop — see [[Open Questions]].
- **Pagination:** Canvas pages results (`per_page`, `Link` headers) — wrapper code must follow pages or silently miss data.
- **Rate limits:** Canvas throttles; the sync loop sleeps between calls ([[Flow - Background Canvas Sync]]).
- **Demo mode:** `achieveup_canvas_demo_service.py` fakes all of this when `ENABLE_DEMO_MODE=true`, so you can develop without a real token.
- Docs: https://canvas.instructure.com/doc/api/
