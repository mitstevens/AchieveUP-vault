---
tags: [flow, knowgap]
---

# Flow: Student Video Recommendations (KnowGap's core loop)

A student misses quiz questions in Canvas → the extension shows them YouTube videos that teach exactly those topics.

```mermaid
sequenceDiagram
    participant S as Student (extension popup)
    participant BE as Backend
    participant DB as MongoDB (Quiz Questions)
    participant AI as OpenAI
    participant YT as YouTube API

    S->>BE: POST /get-assessment-videos<br/>(course, quiz, student token)
    BE->>BE: find student's incorrect questions<br/>(from synced Canvas submission data)
    loop each missed question
        BE->>DB: cached core topic + video?
        alt cache hit
            DB-->>BE: stored video rec
        else cache miss
            BE->>AI: "extract the core topic of this question"
            AI-->>BE: topic string
            BE->>YT: search videos for topic
            YT-->>BE: top result(s)
            BE->>DB: store topic + video on the question doc
        end
    end
    BE-->>S: questions + videos
    S->>S: render list, watch, 👍/👎 vote
```

## Key points
- **Caching is the design center:** the AI topic and chosen video are stored on the question document in the `Quiz Questions` collection, so the expensive OpenAI+YouTube path runs once per question, ever (until refreshed).
- **Incorrect answers** come from quiz submission data the [[Flow - Background Canvas Sync|sync loop]] already pulled — endpoint `/get-incorrect-questions`.
- **Instructor curation:** instructors can override the video for any question from `InstructorView.jsx` → `POST /add-video`.
- **Votes:** students rate videos (`/get-video-votes` + a vote-update path) — intended to improve recommendations later.
- **Support videos** are a parallel, simpler flow: `POST /get-support-video` returns mental-health/wellness videos chosen by the student's **risk level** ([[Flow - Risk Prediction]]) — `support_service.py`.

## Files to read for this flow
Extension: `Studentview.jsx` (fetch + render), `InstructorView.jsx` (curation)
Backend: `video_routes.py` → `video_service.py` → `utils/ai_utils.py` + `utils/youtube_utils.py`; `support_routes.py` → `support_service.py`
