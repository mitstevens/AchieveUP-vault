---
tags: [team, questions]
---

# Open Questions

Issues found while building this vault (June 2026), **then re-verified against the code with exact evidence** before listing here. ✅ = confirmed in code. ⚠️ = depends on deployment state we can't see locally (ask whoever has Heroku access). Resolved items get moved to the bottom with the answer.

## ✅ Confirmed — documentation wrong
- [ ] **Frontend README misdescribes the backend.** `achieveup-frontend/README.md` lines 22–26 say "Backend (Node.js + Express)… Tech Stack: Node.js, Express". The backend is **Python Quart** (`knowgap-backend/app.py` imports `quart`; `requirements.txt`). Other README claims should be treated as unreliable until checked.
- [ ] README lists multiple repo URLs (`dylankatchen/...`, `nsanchez9009/...`, `AndresQ9/...`). **Which forks are canonical** and where do PRs go?
- [ ] Frontend prod URL appears as `achieveup-frontend.netlify.app` (README), `achieveup.netlify.app` + `achieveupapp.com` (CORS list in `app.py`). Which is live?

## ✅ Confirmed — dead/duplicated backend code
- [ ] **`instructor_routes.py` (17 routes under `/instructor/*`) has zero callers.** Verified: no frontend, extension, or backend test references any `/instructor/...` path. The frontend uses the `/achieveup/instructor/*` versions in `achieveup_routes.py`. → candidate for deletion after team confirms no other client exists.
- [ ] **Route path collision:** BOTH `instructor_routes.py` and `achieveup_routes.py` define `/instructor/analyze-questions-with-ai` and `/instructor/bulk-assign-skills-with-ai`. Two registered rules for the same path — one silently shadows the other. No callers either way, but it's a landmine.
- [ ] **~10 dead functions in frontend `api.ts`** point at endpoints that don't exist in the backend (`/achieveup/instructor/courses/suggest-skills`, `.../questions/analyze-ai`, `.../skills/bulk-assign-ai`, `.../assessment/evaluate`, `.../badges/generate-web-linked`, `/achieveup/badges/course/<id>`, `/canvas/instructor/quizzes/<id>/submissions`, `.../enrollment`, `.../validate-token`, canvasInstructorAPI's `/canvas/instructor/quizzes/<id>/questions`). **Verified harmless**: none are called from any component — every live screen uses endpoints that exist. → cleanup, not a bug.
- [ ] Many overlapping analytics/progress collections in `config.py` — which are actually written/read today? (Not yet audited.)

## ✅ Confirmed — Canvas host inconsistency (potential real bug for UCF)
- [ ] The AchieveUp side has **no per-user Canvas host**. All AchieveUp Canvas calls (incl. mastery sync in `canvas_submissions_service.py`) use one global `CANVAS_API_URL` = env var, defaulting to `https://canvas.instructure.com/api/v1` (`config.py:62`).
- [ ] Meanwhile the sync loop's AchieveUp pass hardcodes `link = "canvas.instructure.com"` (`app.py:174`) for the *legacy* update functions it also runs. So if Heroku sets `CANVAS_API_URL=https://webcourses.ucf.edu/api/v1`, **the two halves of the same sync pass talk to different Canvas instances**; if it doesn't, UCF instructors' AchieveUp data never syncs from Webcourses at all. ⚠️ **Ask: what is `CANVAS_API_URL` set to on Heroku?**

## ⚠️ Deployment-state questions (code fact confirmed, prod state unknown)
- [ ] `ACHIEVEUP_JWT_SECRET` has a hardcoded fallback `"achieveup-secret-key-change-in-production"` (`config.py:61`). If the env var is unset on Heroku, login tokens are forgeable. **Ask/check Heroku config.**
- [ ] Is the Heroku dyno always-on? The 10-min sync loop runs *inside* the web process (`app.py` `before_serving`) and stops whenever the dyno sleeps.
- [ ] Is there real instructor/student data in the prod DB (FERPA implications for testing)?

## ✅ Confirmed — extension repo housekeeping
- [ ] Root-level `manifest.json` + `index.html` + `popup.js` are a **standalone "YouTube Video Finder" demo** unrelated to the product (verified: root index.html is a YouTube search page; webpack builds the real extension only from `src/pages/*`). Why is it in the repo / safe to delete?
- [ ] Is the extension published on the Chrome Web Store, or distributed as `build.zip`?
