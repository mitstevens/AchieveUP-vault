---
tags: [overview, reference]
---

# Glossary

Domain terms you'll see constantly in code, comments, and collection names.

| Term | Meaning |
|---|---|
| **Canvas / Webcourses** | The LMS (Learning Management System). UCF's instance is `webcourses.ucf.edu`; the generic cloud one is `canvas.instructure.com`. All course/quiz/student data originates here. See [[Canvas LMS API]]. |
| **Canvas API token** | A personal access token a user generates in Canvas settings. The backend uses it to call Canvas *on the user's behalf*. Stored **encrypted** (Fernet) in MongoDB. |
| **Skill Matrix** | AchieveUp concept: the list of skills an instructor defines for a course (e.g. "Algorithm Design", "SQL Joins"). Stored in `AchieveUp_Skill_Matrices`. One course can have multiple matrices. |
| **Skill Assignment** | Mapping of a quiz **question → one or more skills**. Done manually or via AI bulk-assign. Stored in `AchieveUp_Question_Skills`. |
| **Mastery** | A student's computed proficiency in a skill, derived from how they answered questions tagged with that skill. Levels: `beginner` / `intermediate` / `advanced`. See `services/mastery_service.py`. |
| **Risk level** | `low` / `medium` / `high` — likelihood a student is struggling/failing. Computed from completion rate, scores, activity. Used by both KnowGap (extension instructor view) and AchieveUp dashboards. See [[Flow - Risk Prediction]]. |
| **Badge** | Gamified credential a student earns for skill milestones (e.g. 80%+ mastery). Has level + shareable public page (`/badges/:studentId` in frontend). |
| **Core topic** | KnowGap concept: the AI-extracted topic of a quiz question, used as the YouTube search query for video recommendations. Cached per question. |
| **Support videos** | Mental-health/wellness videos served by risk level (KnowGap extension third page). `services/support_service.py`. |
| **Legacy token** | Document in the `Tokens` collection from the original KnowGap flow: a user's encrypted Canvas token + their course IDs. The sync loop iterates these. |
| **Instructor portal** | The AchieveUp web app ([[Frontend Overview]]) — instructor-only today; student view is planned. |
| **Demo mode** | Backend feature flag (`ENABLE_DEMO_MODE`) + `achieveup_canvas_demo_service.py` that serves fake Canvas data so the app can be demoed without real tokens. Several `create_demo_*.py` scripts seed it. |
| **Force sync** | Manual trigger (`POST /achieveup/instructor/course/<id>/force-sync`) of the same pipeline the [[Flow - Background Canvas Sync|background loop]] runs, used by the "Sync Now" button on the Progress page. |
| **MOC** | "Map of Content" — Obsidian convention for an index note like [[Home]]. |
