---
tags: [overview]
---

# Project Overview

This project is really **two products sharing one backend**, both built on top of [[Canvas LMS API|Canvas]] (the LMS used at UCF as "Webcourses"):

## 1. KnowGap (the original product)
> *"Course Video Recommendations & Student Risk Prediction"*

- Lives in the **Chrome extension** ([[Extension Overview]]) that students/instructors use *inside Canvas*.
- **For students:** when you get quiz questions wrong, it recommends YouTube videos targeted at the topic of each missed question. Also surfaces mental-health support videos keyed to your risk level.
- **For instructors:** predicts which students are **at risk** of failing, based on quiz performance, so they can intervene early. See [[Flow - Risk Prediction]].
- AI is used to extract the "core topic" of a quiz question (OpenAI), then YouTube is searched for matching videos. Results are cached in MongoDB so lookups aren't repeated. See [[Flow - Student Video Recommendations]].

## 2. AchieveUp (the newer layer)
> *"AI-Powered Skill Tracking"* — instructor portal at achieveup.netlify.app

- Lives in the **React web app** ([[Frontend Overview]]).
- Instead of grades, instructors track **specific skills**: they define a **skill matrix** for a course, map quiz questions to skills (with AI help), and the system computes each student's **mastery** per skill from their Canvas quiz submissions.
- Students earn **badges** for skill milestones (public/shareable badge pages exist).
- See [[Flow - Instructor Skill Workflow]] — this is the core workflow of the whole product.

## The shared backend
Both products are served by **one Python backend** ([[Backend Overview]], `knowgap-backend`), deployed on Heroku at `gen-ai-prime-3ddeabb35bd7.herokuapp.com`. It contains:
- Legacy **KnowGap routes** (videos, risk, support) used by the extension
- Newer **AchieveUp blueprints** (auth, skills, badges, progress, analytics) used by the web app
- A [[Flow - Background Canvas Sync|background sync loop]] that pulls fresh quiz data from Canvas every 10 minutes

## Who built this / who runs it
- Previous UCF Senior Design team(s); now handed to your team.
- Frontend originally by Nico Sanchez, backend by Andres Q (per README).
- Deployments: Netlify (frontend, auto-deploy from GitHub), Heroku (backend), MongoDB Atlas (database).

## Key facts at a glance
| Thing | Value |
|---|---|
| Backend prod URL | `https://gen-ai-prime-3ddeabb35bd7.herokuapp.com` |
| Frontend prod URL | `https://achieveup.netlify.app` (also `achieveupapp.com`) |
| Database | MongoDB Atlas — DB `KnowGap` (prod) / `KnowGap_Dev` (dev) |
| Backend local port | 5001 |
| Frontend local port | 3000 |
| AI | OpenAI (topic extraction, skill suggestion) + YouTube Data API |

Next: [[System Architecture]]
