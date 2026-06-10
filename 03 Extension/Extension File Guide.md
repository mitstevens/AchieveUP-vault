---
tags: [extension, reference]
repo: knowgap-extension
---

# Extension File Guide

## Repo root
| File | Purpose |
|---|---|
| `manifest.json`, `index.html`, `popup.js`, `popup.css` | ⚠️ Standalone **demo** extension (YouTube search with your own API key). Not the product — see warning in [[Extension Overview]]. |
| `package.json` | Scripts: `npm start` (dev server w/ hot reload), `npm run build` (webpack → `build/`). |
| `webpack.config.js` | Builds each page (popup, options, background, content…) as its own bundle; injects env vars (incl. `BACKEND_URL`). |
| `utils/env.js`, `utils/webserver.js`, `utils/build.js` | Boilerplate build tooling (env defaults, dev server, prod build script). |
| `build.zip`, `build (2).zip` | Packaged releases (for Chrome Web Store upload). Generated artifacts — could be gitignored. |
| `tsconfig.json`, `tailwind.config.js`, `postcss.config.js` | TS/Tailwind available, but main views are plain JSX + CSS. |
| `icons/` | Toolbar/store icons. |

## `src/`
| File | Purpose |
|---|---|
| `manifest.json` | **The real MV3 manifest** ("KnowGap for Canvas" v1.3.8): popup, options page, background service worker, content script on Canvas domains, host permissions for Canvas + Heroku backend + localhost. |
| `assets/img/` | Logo/icons bundled into the build. |
| `containers/Greetings/` | Boilerplate sample component (unused). |

## `src/pages/Popup/` — ⭐ the product UI
| File | Purpose |
|---|---|
| `index.jsx` / `index.html` | Mounts the popup React app. |
| `Popup.jsx` (360) | **Controller.** Gets active-tab URL → extracts Canvas base URL + course ID; validates Canvas token against Canvas's own API; syncs course to backend (`/update-course-db`, `/update-course-context`); decides Student vs Instructor view; feedback-form link. |
| `Studentview.jsx` (1206) | **Student experience:** fetches missed quiz questions + recommended videos (`/get-assessment-videos`), video voting, support/mental-health videos by risk (`/get-support-video`), grade display. |
| `InstructorView.jsx` (1519) | **Instructor experience:** class risk overview (`/get-student-grade`-type data, risk toggle), per-question video curation (`/add-video`), course sync controls. |
| `*.css` | Styling for the above. |

## Other `src/pages/` (mostly template boilerplate)
| Dir | Status |
|---|---|
| `Background/` | `index.js` + `background.js` — service worker; minimal logic. |
| `Content/` | `index.js`, `content.js`, `content.styles.css`, `modules/print.js` — injected into Canvas pages. |
| `Options/` | Options page (`Options.tsx`). |
| `Newtab/`, `Panel/`, `Devtools/` | Unused boilerplate from `chrome-extension-boilerplate-react`. Safe to ignore (or delete someday). |

> [!tip] Reading order
> `src/manifest.json` → `Popup.jsx` → skim `Studentview.jsx` top-to-bottom following one feature (e.g. assessment videos) → [[Flow - Student Video Recommendations]] to see the backend half.
