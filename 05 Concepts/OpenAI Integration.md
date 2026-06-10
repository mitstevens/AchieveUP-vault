---
tags: [concept, integration, ai]
---

# OpenAI Integration

Two distinct AI jobs in the backend, both using the OpenAI API (`OPENAI_KEY` env var):

## 1. Core-topic extraction (KnowGap, legacy)
- **Where:** `utils/ai_utils.py`, called from `services/video_service.py`.
- **Job:** given a quiz question's text, return a short topic ("binary search trees") to use as a **YouTube search query**.
- **Cached** on the question doc in `Quiz Questions` so each question is processed once. See [[Flow - Student Video Recommendations]].

## 2. Skill suggestion & question classification (AchieveUp)
- **Where:** `services/achieveup_ai_service.py` (652 lines).
- **Jobs:**
  - *Suggest skills* for a course from name/code/description → seeds the skill matrix.
  - *Analyze questions* → which skills does each quiz question test, plus complexity.
  - *Bulk-assign* → map an entire quiz's questions to matrix skills in one go.
- **Fallback:** if the API fails/unavailable, `COURSE_CODE_MAPPINGS` provides canned skills by course prefix (`COP` → Programming Fundamentals, `CDA` → Computer Architecture, …). Good defensive pattern to preserve demos.
- The master doc mentions the model name in passing; check `achieveup_ai_service.py` for the current model string before assuming.

## Things to keep in mind
- **Cost & latency:** every uncached AI call costs money and ~seconds. Caching (topics) and bulk endpoints exist for this reason — keep that discipline when adding features.
- **Non-determinism:** same question can yield different skills run-to-run; the UI treats AI output as *suggestions* the instructor confirms, which is the right design.
- **Prompts live in code** as f-strings in the two files above — if recommendations feel off, prompt text is the first thing to inspect.

Related: [[Flow - Instructor Skill Workflow]] · [[Flow - Student Video Recommendations]]
