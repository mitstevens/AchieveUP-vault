---
tags: [backend, reference, database]
repo: knowgap-backend
file: config.py
---

# Database Collections

MongoDB Atlas. Database name: **`KnowGap`** in production, **`KnowGap_Dev`** when `ENVIRONMENT=development`. All collection names are constants in `config.py` — never hardcode a collection string elsewhere.

See [[MongoDB Notes]] for how documents/collections work if you're new to Mongo.

## Legacy KnowGap collections
| Collection | Holds |
|---|---|
| `Tokens` | Encrypted Canvas tokens + course IDs per user (the sync loop iterates this) |
| `Quiz Questions` | Quiz questions per course **+ cached core topic & video recs** (indexed on `course_id`) |
| `Students` | Student quiz performance data |
| `Courses` | Course metadata |
| `Course Contexts` | Course context (used for topic generation / risk toggle) |

## AchieveUp collections (the important ones)
| Collection | Holds |
|---|---|
| `AchieveUp_Users` | Instructor accounts: email, password hash, role, **encrypted Canvas token**, token type |
| `AchieveUp_Skill_Matrices` | Skill matrices: `{ course_id, instructor_id, matrix_name, skills[] }` |
| `AchieveUp_Question_Skills` | Question→skills mappings per course |
| `AchieveUp_Skill_Assignments` | Skill assignment records (related to above) |
| `AchieveUp_Student_Skill_Mastery` | Computed mastery per student/skill — output of `mastery_service.py` |
| `AchieveUp_Progress`, `AchieveUp_User_Progress` | Student progress documents |
| `AchieveUp_Badges`, `AchieveUp_User_Badges`, `AchieveUp_Badge_Progress` | Badge definitions, earned badges, progress toward badges |
| `AchieveUp_Analytics`, `AchieveUp_Course_Analytics`, `AchieveUp_Student_Analytics`, `AchieveUp_Skill_Analytics`, `AchieveUp_Progress_Analytics` | Cached/aggregated analytics |
| `AchieveUp_Canvas_Courses`, `AchieveUp_Canvas_Quizzes`, `AchieveUp_Canvas_Questions`, `AchieveUp_Quiz_Submissions` | Cached Canvas data (so the UI doesn't hit Canvas live every time) |
| `AchieveUp_Course_Descriptions` | Instructor-written course descriptions (AI context). Unique index on `(course_id, instructor_id)` — created in `app.py`. |
| `AchieveUp_Import_Status` | Tracks matrix/assignment imports between courses |

## Observations
- There are **a lot** of analytics/progress collections with overlapping names — before adding a new one, check which are actually written/read (grep the constant in `services/`). Some may be vestigial. Tracked in [[Open Questions]].
- No formal schema/migrations — Mongo is schemaless; the de-facto schema lives in the service code that writes each collection. `types/index.ts` in the frontend is the closest thing to a documented data model.
- Sensitive data: Canvas tokens are Fernet-encrypted (`utils/encryption_utils.py`); passwords hashed in `achieveup_auth_service.py`.
