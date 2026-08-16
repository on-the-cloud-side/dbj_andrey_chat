# Skogschatt

A bilingual (SV/EN) community chat PWA with two visual "vibes":
- **Skog** — Swedish forest, dimmed forest-path SVG background
- **Kikinda** — dimmed long-eared owl portrait SVG background (Kikinda, Serbia, is famous for its owl colony)

Users belong to a `community` (currently `MALMO` and `KIKINDA`; Andrey is in
MALMO, Dusan is in KIKINDA). Chat is a single shared stream; each message is
shown with the sender's name prefixed by their community code, e.g.
"KIKINDA Dusan". More communities/users can be added later — see Conventions.

## Conversation protocol

- Prose Quantity != Prose Quality
  - Be very brief in your answers 
  - If you think longer answer is required, only then make it longer
- Use simple terminology
  - Move explaining of complex and necessary stuff into footnote section of the document
    - Call it "Vocabulary"
- Do not over explain "in line"
  - Instead use the "Vocabulary" section
    - And point to external sources if any
- Do not assume anything, if in doubt ask

## Stack
- Vanilla JS (no React, no framework, no build step)
- Firebase Realtime Database for chat + Google Sign-In auth (config in `src/app.js`)
- Static PWA, deployed via GitHub Pages from `master` branch root

## Layout
```
public/
  manifest.json
  images/forest-path.svg
  images/kikinda-owls.svg
src/
  index.html   - entry point (markup only, loads index.js as a module)
  index.js     - app bootstrap: Firebase init, auth, sign-in/out, profile lookup
  tests.js     - sanity checks run after sign-in, before app logic; logs via
                 console.log("[tests] ..."); index.js halts if these fail
  app.js       - Firebase config
  chat.js      - Firebase realtime chat logic
  ui.js        - vibe switching (Skog/Kikinda) + language toggle
  forest.css   - all styling, including vibe backgrounds
  SKOGCHATT.js - sets window.SKOGCHATT.build_stamp (local time), update
                 on every change so you can verify the deploy in the console
service-worker.js - PWA cache (bump CACHE version when assets change)
database.rules.json - RTDB security rules (any signed-in Google account)
```

## Conventions
- Keep it vanilla JS — no frameworks.
- Background images live in `public/images/` and are applied via CSS
  `background-image` with a dark `linear-gradient` overlay for dimming.
  Don't bake dimming into the SVGs themselves — keep the overlay in `forest.css`.
- All paths must stay relative (GitHub Pages serves from `/skogschatt/` subpath).
- All devices load the same deployed `app.js` — there is no per-device
  config file. User identity/access is data, not code:
  - `skogschatt/users/{ordinal} -> { community, name, email }` — plain
    ordinal keys (`1`, `2`, ...). The app fetches all of `skogschatt/users`
    after sign-in and finds the entry whose `email` matches the signed-in
    Google account, to build the display label (e.g. "KIKINDA Dusan").
  - `skogschatt/messages` read/write requires only `auth != null` — any
    signed-in Google account can chat. `skogschatt/users` is purely for
    display (community/name lookup), not an access-control allowlist.
  - Adding a new person/community means adding one entry under
    `skogschatt/users` (with `community`, `name`, `email`) in the Firebase
    Console (Realtime Database data, not rules) — no redeploy needed.
- Auth uses `signInWithPopup` for Google Sign-In. (`signInWithRedirect` was
  tried first but failed: the redirect's pending-auth state was lost due to
  third-party storage partitioning between the `firebaseapp.com` authDomain
  and the `github.io` hosting domain.) A login screen (`#login-screen` in
  `index.html`) is shown until `onAuthStateChanged` reports a recognized user.
- Bump `CACHE` in `service-worker.js` whenever cached assets (HTML/CSS/JS/images) change.
- `src/SKOGCHATT.js` sets `window.SKOGCHATT.build_stamp` to the local time
  the file was last edited. It's not in the service worker's precache list,
  so it's always fetched fresh — inspect `window.SKOGCHATT` in the dev
  console to confirm the browser is running the latest deploy.
- After sign-in, `index.js` runs `tests.js`'s `runTests(db, user)` first and
  halts (shows "Unrecognized account") if it returns false — app logic
  (profile lookup, chat init) only runs after tests pass. Don't mix test
  checks into app logic or vice versa. Diagnostics use
  `console.log("[tests] ...")` / `console.log("[index] ...")` so the dev
  console shows what's happening; keep `window.SKOGCHATT` for build-version
  info, not ad-hoc debug dumps.

## Deployment
- Live at: https://dbjarh.github.io/skogschatt/src/index.html
- Push to `master` → GitHub Pages rebuilds automatically.


## Document versioning

- Every markdown file **SHOULD** (not must) carry a decimal `version:` key in its front matter:

```yaml
---
version: 0.1
---
```

- `0.1` .. `1.0` — pre-releases leading up to release 1
- `1.1` .. `2.0` — releases 1.1 through 2.0
- and so on by the same pattern

SHOULD, not MUST: skip it where this repo forbids front matter, and where front matter already exists just add the `version` key without disturbing the rest.
