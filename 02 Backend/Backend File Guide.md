---
tags: [backend, reference]
repo: knowgap-backend
---

# Backend File Guide

Every file in `knowgap-backend` and its job. Line counts included as a "how big a read is this" signal.

## Root
| File | Purpose |
|---|---|
| `app.py` (239) | App wiring: CORS, route registration, Mongo client, index creation, background sync loop. **Read first.** |
| `config.py` (89) | All env vars + **every MongoDB collection name** as constants. See [[Database Collections]]. Fails fast at import if required env vars missing. |
| `requirements.txt` | Python deps: quart, quart-cors, motor, openai, PyJWT, cryptography, hypercorn, etc. |
| `Procfile` | Heroku start command (hypercorn ASGI server). |
| `demo.env` | Template for your `.env`. |
| `privacy-policy.html` | Hosted privacy policy (extension store requirement). |
| `README.md` | KnowGap-era readme (features/TODOs). |
| `BackEndREADME.md` | Same readme, duplicate copy. |
| `ACHIEVEUP_MASTER_DOCUMENTATION.md` (212) | **Best existing doc** — AchieveUp endpoints, schema, deployment summary. Some drift from code. |
| `PRODUCTION_READINESS_SUMMARY.md` | Status checklist from previous team. |
| `test_production_transcript.py` | One-off test script against prod transcript endpoint. |
| `create_demo_*.py`, `demo_data_generator.py`, `create_comprehensive_mock_data.py` | **Seed scripts** for demo accounts/data (various attempts; note `create_demo_ssl_bypass.py` exists because of Atlas TLS issues). Used with demo mode. |
| `docs/` | Coding standard, privacy policy, a frontend-update note. |
| `tests/` | Unit tests for core features. |

## `routes/` — HTTP endpoints (see [[API Route Reference]] for the endpoint table)
| File | Generation | Purpose |
|---|---|---|
| `base_routes.py` (26) | Legacy | `/` health-check root. |
| `user_routes.py` (39) | Legacy | `/get-user`, user lookup for extension. |
| `video_routes.py` (282) | Legacy | Video recommendation endpoints: `/get-course-videos`, `/get-assessment-videos`, `/add-video`, votes, transcripts. |
| `support_routes.py` (22) | Legacy | `/get-support-video` — mental-health videos by risk level. |
| `course_routes.py` (264) | Legacy | Course data sync triggers, student grade/profile, incorrect questions, risk toggle, `/add-token`. |
| `course_utils.py` (155) | Legacy | Helpers shared by course routes (quiz/question fetching). |
| `auth_routes.py` (397) | AchieveUp | `/auth/*` — signup/login/me/profile/password/Canvas-token validation. |
| `canvas_routes.py` (279) | AchieveUp | `/canvas/*` — proxied Canvas data (courses, quizzes, questions) using stored tokens. |
| `skill_routes.py` (281) | AchieveUp | `/achieveup/skills/*` + AI suggest/assign. |
| `badge_routes.py` (407) | AchieveUp | `/badges/*` + `/achieveup/badges/*` — generation, earned lists, public + shareable badges. |
| `progress_routes.py` (164) | AchieveUp | `/achieveup/progress/*` — per-student skill progress get/update. |
| `analytics_routes.py` (391) | AchieveUp | `/analytics/*` — course analytics, risk assessment, trends, export. |
| `achieveup_routes.py` (1887) | AchieveUp | ⚠️ **The kitchen sink.** Matrix CRUD, AI endpoints, instructor dashboard/analytics, force-sync, import/export, course descriptions. Many `instructor_routes.py` near-duplicates. |
| `instructor_routes.py` (915) | AchieveUp | `/instructor/*` — older instructor variants of dashboard/analytics/AI endpoints. Overlaps heavily with `achieveup_routes.py`. |

## `services/` — business logic
| File | Purpose |
|---|---|
| `achieveup_service.py` (2012) | ⚠️ Largest file. Skill matrix logic, skill assignment, instructor dashboard/analytics aggregation, import/export. Backs `achieveup_routes.py`. |
| `achieveup_auth_service.py` (471) | Password hashing, JWT create/verify, user CRUD, Canvas token encryption + validation. |
| `achieveup_canvas_service.py` (806) | All Canvas REST calls for AchieveUp (courses, quizzes, questions, enrollments) + token-type checks. |
| `achieveup_canvas_demo_service.py` (455) | Fake Canvas responses when demo mode is on (`ENABLE_DEMO_MODE`). |
| `achieveup_ai_service.py` (652) | [[OpenAI Integration|OpenAI]] calls: suggest skills for a course, analyze questions, bulk map questions→skills; fallback `COURSE_CODE_MAPPINGS` when AI unavailable. |
| `canvas_submissions_service.py` (595) | Pulls quiz **submissions** from Canvas and updates mastery (`sync_course_submissions_direct`) — the heart of [[Flow - Background Canvas Sync]]. |
| `canvas_submissions_examples.py` (86) | Example payloads/docs for the above. |
| `mastery_service.py` (212) | Converts question results → per-skill mastery scores/levels. Core algorithm of AchieveUp. |
| `progress_service.py` (357) | Reads/writes student skill progress documents. |
| `badge_service.py` (601) | Badge generation rules, earned-badge queries, shareable/public badges. |
| `analytics_service.py` (1521) | Risk assessment (completion 40% / score 30% / activity 30%), course & student analytics, trends, export. See [[Flow - Risk Prediction]]. |
| `course_service.py` (629) | Legacy KnowGap: update student quiz data & quiz questions per course from Canvas. |
| `video_service.py` (388) | Legacy KnowGap: core-topic generation + YouTube lookup + caching per question. See [[Flow - Student Video Recommendations]]. |
| `support_service.py` (130) | Mental-health support video selection by risk level. |
| `user_service.py` (39) | Legacy user lookup. |
| `skill_service.py` (345) | Skill assignment storage/queries (question↔skills). |

## `utils/`
| File | Purpose |
|---|---|
| `db_utils.py` | Shared Mongo client/collection accessors. |
| `encryption_utils.py` | Fernet encrypt/decrypt for Canvas tokens (`HEX_ENCRYPTION_KEY`). |
| `ai_utils.py` | Legacy OpenAI helper (core topic extraction for videos). |
| `youtube_utils.py` | YouTube Data API search wrapper. |
| `course_utils.py` | Canvas course/quiz fetch helpers (legacy side). |

> [!tip] Where do I start reading?
> `app.py` → `config.py` → `routes/skill_routes.py` (small, typical blueprint) → `services/mastery_service.py` (small, core algorithm). Avoid starting with `achieveup_routes.py` or `achieveup_service.py` — they're the biggest and messiest.
