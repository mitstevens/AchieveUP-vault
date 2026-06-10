---
tags: [concept, extension]
---

# Chrome Extension Concepts (Manifest V3)

A Chrome extension is a zip of web pages + scripts that Chrome runs in special contexts. **Manifest V3** is the current platform version; `manifest.json` declares everything.

## The contexts (and where this project uses each)
| Context | What it is | In this project |
|---|---|---|
| **Popup** | Small page shown when you click the toolbar icon. Dies when closed — no persistent state. | `src/pages/Popup/` — the entire KnowGap UI lives here |
| **Content script** | JS injected *into* matching web pages; can read/modify the page DOM but runs in an isolated world. | `src/pages/Content/` — injected into Canvas pages |
| **Background service worker** | Event-driven script with no UI; Chrome starts/stops it on demand (MV3 killed persistent background pages). | `src/pages/Background/` |
| **Options page** | Settings UI at chrome://extensions → Details. | `src/pages/Options/` |

## Key manifest fields (see `src/manifest.json`)
- `permissions` / `host_permissions` — which origins the extension may call (Canvas hosts, the Heroku backend, localhost). Adding a new backend URL requires a manifest change + re-release.
- `content_scripts.matches` — which pages get the injected script (`canvas.instructure.com/*`, `webcourses.ucf.edu/*`).
- `web_accessible_resources` — files the host page may load (`sidebar.html`).
- `content_security_policy` — restricts script sources inside extension pages.

## APIs used in this code
```js
chrome.tabs.query({active: true, currentWindow: true}, ...)  // Popup.jsx: get the Canvas tab's URL
chrome.storage.sync.get/set(...)                             // persist small settings across devices
chrome.runtime.id                                            // extension's own ID (used in Origin header)
```

## Build pipeline quirk
The popup is **React**, so it must be compiled — webpack bundles each context into `build/` (`popup.bundle.js`, `contentScript.bundle.js`, `background.bundle.js`…). You load the **build/ folder**, not `src/`, via chrome://extensions → Load unpacked. `BACKEND_URL` is baked in at build time, so switching between localhost and Heroku means rebuilding.

## Debugging tips
- Popup devtools: right-click inside the popup → Inspect (it's just a page).
- Content script logs appear in the **host page's** console; background logs in the service-worker inspector on chrome://extensions.
- CORS: the backend explicitly allows `chrome-extension://*` origins (see `app.py`).

Related: [[Extension Overview]] · [[Extension File Guide]]
