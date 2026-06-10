---
tags: [team]
---

# Onboarding Checklist

Work through this top-to-bottom in your first week or two. Check items off as you go (they're real checkboxes in Obsidian).

## Understand the product (Day 1)
- [ ] Read [[Project Overview]] and [[System Architecture]]
- [ ] Skim [[Glossary]]
- [ ] Visit the live frontend: https://achieveup.netlify.app (make a test account if signup is open)
- [ ] Read [[Flow - Instructor Skill Workflow]] — the core loop

## Get access (ask teammates / sponsor)
- [ ] GitHub access to all 3 repos (and find out which forks are canonical — see [[Open Questions]])
- [ ] MongoDB Atlas access (or at least the dev connection string)
- [ ] Heroku app access (`gen-ai-prime-3ddeabb35bd7`) — needed for logs and env vars
- [ ] Netlify access
- [ ] `.env` values for the backend (`DB_CONNECTION_STRING`, `HEX_ENCRYPTION_KEY`, `OPENAI_KEY`, `YOUTUBE_API_KEY`)
- [ ] A Canvas API token of your own (Webcourses → Account → Settings → New Access Token) — note: student tokens see less than instructor tokens

## Run things locally
- [ ] Backend: venv → `pip install -r requirements.txt` → `.env` from `demo.env` → `python app.py` → check http://localhost:5001/
- [ ] Frontend: `npm install` → `npm start` → log in against your local backend (`REACT_APP_API_URL=http://localhost:5001`)
- [ ] Extension: `npm install` → `npm run build` → load `build/` unpacked in Chrome → open a Canvas course page → click the icon
- [ ] Try demo mode (`ENABLE_DEMO_MODE=true` + one of the `create_demo_*.py` seeders) if you lack real Canvas data

## Read code with the vault open
- [ ] Backend: `app.py` → `config.py` → `skill_routes.py` → `mastery_service.py` (use [[Backend File Guide]])
- [ ] Frontend: `App.tsx` → `AuthContext.tsx` → `api.ts` → `SkillMatrixCreator.tsx` (use [[Frontend File Guide]] + [[React Concepts]])
- [ ] Extension: `src/manifest.json` → `Popup.jsx` (use [[Extension File Guide]])
- [ ] Open MongoDB Compass and browse the collections against [[Database Collections]]

## First contributions (low-risk ideas)
- [ ] Fix the frontend README's wrong claim that the backend is Node/Express
- [ ] Add `.gitignore` entries for `build.zip` artifacts in the extension repo
- [ ] Pick one item from [[Open Questions]] and resolve it
- [ ] Update this vault wherever the code proved a note wrong
