---
tags: [overview, architecture]
---

# System Architecture

Three repos, one backend, two user-facing clients. Everything ultimately revolves around **Canvas quiz data** stored in **MongoDB**.

```mermaid
flowchart TB
    subgraph Clients
        FE["achieveup-frontend<br/>(React + TS, Netlify)<br/>Instructor portal"]
        EXT["knowgap-extension<br/>(Chrome MV3 extension)<br/>Runs inside Canvas pages"]
    end

    subgraph Backend["knowgap-backend (Quart, Heroku)"]
        ROUTES["routes/ — HTTP endpoints"]
        SERVICES["services/ — business logic"]
        SYNC["Background sync loop<br/>(every 10 min)"]
    end

    subgraph External["External services"]
        CANVAS["Canvas LMS API<br/>(webcourses.ucf.edu)"]
        OPENAI["OpenAI API<br/>(topic & skill AI)"]
        YT["YouTube Data API<br/>(video search)"]
    end

    DB[("MongoDB Atlas<br/>DB: KnowGap")]

    FE -->|"REST + JWT"| ROUTES
    EXT -->|"REST + Canvas token"| ROUTES
    ROUTES --> SERVICES
    SERVICES --> DB
    SERVICES --> CANVAS
    SERVICES --> OPENAI
    SERVICES --> YT
    SYNC --> SERVICES
    EXT -.->|"reads page URL, calls Canvas directly for token validation"| CANVAS
```

## How the pieces map to repos

| Repo | What it is | Talks to | Details |
|---|---|---|---|
| `achieveup-frontend` | React SPA for **instructors** (AchieveUp) | Backend only | [[Frontend Overview]] |
| `knowgap-backend` | Quart async API — **single source of truth** | MongoDB, Canvas, OpenAI, YouTube | [[Backend Overview]] |
| `knowgap-extension` | Chrome extension for **students & instructors inside Canvas** (KnowGap) | Backend + Canvas directly | [[Extension Overview]] |

## Two route "generations" in the backend
This is the most important structural fact to understand early:

1. **Legacy KnowGap routes** — registered with `init_*_routes(app)` functions, flat endpoints like `/get-course-videos`, `/get-student-grade`, `/add-token`. Used by the **extension**.
2. **AchieveUp blueprints** — Quart Blueprints (`auth_bp`, `skill_bp`, `badge_bp`, …) with prefixed endpoints like `/achieveup/matrix/create`, `/auth/login`. Used by the **web frontend**.

Both generations live in `app.py` side by side. See [[Backend File Guide]].

## Authentication differs per client
- **Frontend → backend:** email/password signup, [[JWT Authentication|JWT]] in `Authorization: Bearer`, Canvas API token stored encrypted server-side. See [[Flow - Authentication]].
- **Extension → backend:** sends the user's **Canvas API token** with requests (older cookie/token model, no JWT account).

## Data backbone
MongoDB collections are defined centrally in `config.py` — see [[Database Collections]]. The [[Flow - Background Canvas Sync|sync loop]] keeps quiz/submission data fresh; everything else (mastery, risk, analytics, badges) is computed from it.

## Deployment
```mermaid
flowchart LR
    GH["GitHub repos"] -->|auto-deploy| NETLIFY["Netlify (frontend)"]
    GH -->|"git push heroku / pipeline"| HEROKU["Heroku (backend, Procfile: hypercorn)"]
    EXTZIP["build.zip"] -->|manual upload| CWS["Chrome Web Store / load unpacked"]
```
