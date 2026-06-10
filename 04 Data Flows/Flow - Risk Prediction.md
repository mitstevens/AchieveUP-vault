---
tags: [flow, knowgap, achieveup]
---

# Flow: Risk Prediction

Both products surface a per-student **risk level** (`low` / `medium` / `high`) so instructors can intervene before a student fails.

## The algorithm (per `analytics_service.py` and master doc)
Weighted multi-factor score:
| Factor | Weight |
|---|---|
| Completion rate (how much assigned work was done) | 40% |
| Average score on quizzes | 30% |
| Activity frequency (recency/regularity of submissions) | 30% |

The combined score is bucketed into low/medium/high. Implementation: `services/analytics_service.py` (1,521 lines — search for "risk"). Endpoint: `GET /analytics/course/<id>/risk-assessment`.

> [!note] The exact thresholds and formula details live only in the code — the previous team's own TODO list says "provide detailed breakdowns of how the risk score is calculated." Good candidate for your team to document or improve.

## Where risk shows up
1. **Extension instructor view** (`InstructorView.jsx`): class-wide risk table inside Canvas. There's a per-course on/off switch: `GET /get-toggle-risk/<course_id>`.
2. **Extension student view**: risk level selects which **mental-health support videos** the student sees (`/get-support-video`).
3. **Web app Progress page**: each student row shows a risk chip (green/yellow/red) — data arrives inside `/achieveup/instructor/courses/<id>/student-analytics`.

## Data dependencies
Risk needs fresh submission data, so it's only as current as the last [[Flow - Background Canvas Sync|sync]]. No external ML — it's a hand-tuned heuristic, not a trained model (despite "prediction" branding).

Related: [[Flow - Instructor Skill Workflow]] · [[Flow - Student Video Recommendations]]
