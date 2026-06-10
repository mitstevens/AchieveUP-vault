---
tags: [backend, reference, api]
repo: knowgap-backend
---

# API Route Reference

All backend endpoints, grouped by purpose. Extracted from `routes/*.py` (June 2026). When in doubt, grep the route string in `routes/`.

## Auth (`auth_routes.py`) — used by web frontend
| Method | Path | Purpose |
|---|---|---|
| POST | `/auth/signup` | Create instructor account (optionally with Canvas token) |
| POST | `/auth/login` | Email/password → JWT |
| GET | `/auth/me` / `/auth/verify` | Current user from JWT |
| PUT | `/auth/profile` / `/auth/password` | Update profile / change password |
| POST | `/auth/validate-canvas-token` | Check a Canvas token works & detect type |
| GET | `/auth/token-status` | Canvas token presence/health |
| POST | `/auth/refresh-token` | New JWT |

## Canvas proxy (`canvas_routes.py`)
| Method | Path |
|---|---|
| GET | `/canvas/courses`, `/canvas/courses/<id>/quizzes`, `/canvas/quizzes/<id>/questions` |
| GET | `/canvas/instructor/courses`, `/canvas/instructor/courses/<id>/quizzes`, `/canvas/instructor/courses/<id>/quizzes/<qid>/questions` |
| GET | `/canvas/test-connection` |

## Skill matrices & assignments (`skill_routes.py`, `achieveup_routes.py`)
| Method | Path | Purpose |
|---|---|---|
| POST | `/achieveup/matrix/create` | Create skill matrix |
| GET | `/achieveup/matrix/<course_id>` · `/achieveup/matrix/course/<course_id>` | Get matrix / all matrices for course |
| PUT/DELETE | `/achieveup/matrix/<matrix_id>` · `/achieveup/matrix/delete/<matrix_id>` | Update / delete |
| POST | `/achieveup/matrix/import`, `/achieveup/skills/import` | Copy matrices/assignments from another course |
| GET | `/achieveup/import-status/<course_id>` | Was anything imported? |
| POST | `/achieveup/skills/assign` | Save question→skills mapping |
| GET | `/achieveup/skills/assignments` | Fetch mappings for question IDs |
| POST | `/achieveup/skills/suggest`, `/achieveup/ai/suggest-skills` | AI skill suggestions |
| POST | `/achieveup/ai/analyze-questions`, `/achieveup/questions/analyze` | AI question analysis |
| POST | `/achieveup/ai/bulk-assign` | AI maps a whole quiz to skills |
| GET/PUT | `/achieveup/course-description/<course_id>` | Course description (AI context) |

## Progress, badges, analytics (AchieveUp)
| Method | Path | Purpose |
|---|---|---|
| GET/PUT | `/achieveup/progress/<student_id>/<course_id>` | Skill progress per student+course |
| POST | `/achieveup/progress/update` | Write progress |
| POST | `/achieveup/badges/generate`, `/badges/generate` | Generate badges from mastery |
| GET | `/achieveup/badges/<student_id>`, `/achieveup/badges/student/<sid>/earned` | Student badges |
| GET | `/achieveup/public/badges/student/<sid>/earned` | **No auth** — public badge page |
| GET | `/badges/<id>/verify`, POST `/badges/<id>/share` | Badge verification/sharing |
| GET | `/achieveup/graphs/individual/<student_id>` | Graph data |
| GET | `/achieveup/export/<course_id>`, POST `/achieveup/import` | Course data export/import |
| GET | `/analytics/course/<id>`, `/analytics/course/<id>/risk-assessment`, `/analytics/course/<id>/students`, `/analytics/trends`, `/analytics/compare`, `/analytics/export…` | Analytics suite |

## Instructor dashboards (two overlapping flavors ⚠️)
| Flavor | Examples |
|---|---|
| `/achieveup/instructor/*` (in `achieveup_routes.py`) — **what the frontend actually calls** | `/achieveup/instructor/dashboard`, `/achieveup/instructor/courses/<id>/student-analytics`, `/achieveup/instructor/course/<id>/force-sync`, `/achieveup/instructor/skill-matrix/create` |
| `/instructor/*` (in `instructor_routes.py`) — older duplicates | `/instructor/courses`, `/instructor/courses/<id>/analytics`, `/instructor/ai/suggest-skills`, `/instructor/bulk-assign-skills-with-ai` |

> ✅ Verified June 2026: the 17 `/instructor/*` routes have **zero callers** (no frontend, extension, or test references). Also note both files register `/instructor/analyze-questions-with-ai` and `/instructor/bulk-assign-skills-with-ai` — a path collision where one handler silently shadows the other. Details in [[Open Questions]].

## Legacy KnowGap (used by the extension)
| Method | Path | Purpose |
|---|---|---|
| POST | `/add-token` | Store encrypted Canvas token + course list (`Tokens` collection) |
| POST | `/update-course-db`, `/update-course-context` | Popup-triggered course sync / context update (`course_routes.py`) |
| POST | `/sync-all-quizzes-questions` | Re-pull all quizzes+questions for a course |
| POST | `/get-course-videos`, `/get-assessment-videos` | Video recs for course / one quiz |
| POST | `/add-video`, `/remove-video`, `/update-video-link`, `/update-course-videos`, `/update-all-videos` | Instructor video curation & refresh |
| POST | `/get-video-votes`, `/vote-video`, `/set-video-watched`, `/get-video-transcript` | Voting, watched-state, transcripts |
| POST | `/get-incorrect-questions` | Student's missed questions |
| POST | `/get-student-grade`, `/get-student-profile` | Grade/profile via Canvas |
| POST | `/get-support-video` | Mental-health video by risk level |
| GET | `/get-toggle-risk/<course_id>` · POST `/update-toggle-risk/<course_id>` | Risk display on/off per course |
| POST | `/get-course-quizzes`, `/get-questions-by-course/<id>` | Quiz/question lists |
| POST | `/get-user` | User lookup |
| GET | `/` | Health check |

> ✅ Verified June 2026: every endpoint the extension actually calls (15 of them) exists in the backend.

Related: [[Frontend API Layer]] (client side) · [[Backend File Guide]]
