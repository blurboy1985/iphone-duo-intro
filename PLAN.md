# iPhone Duo — 3D intro (HTML)

**Task:** A single-file HTML page that plays a cinematic, real-time 3D intro for the foldable iPhone Duo (announced Sep 9 2026).
The user wrote "iPhone Dual"; assumed to mean **iPhone Duo**.

## Facts used (sources: apple.com newsroom, MacRumors roundup)
- Book-style foldable. 5.4" outer display, 7.6" inner display. Folded ~9.5 mm, open ~4.5 mm ("thinnest iPhone ever").
- Grade 5 titanium, mirror-polished. Finishes: Night Sky, Star White.
- Dual 48MP Fusion rear cameras, A20 Pro, Touch ID power button.
- From $1,999. Pre-order Oct 16, available Oct 23 2026.

## Done =
- `index.html` opens in a browser and plays a ~17 s intro with no console errors.
- The phone model folds and unfolds on a hinge. The outer and inner screens light up.
- The intro ends on a title and launch dates. Then the viewer can replay, switch finish, fold or unfold, and drag to rotate.
- Works at desktop and phone widths. Respects `prefers-reduced-motion`.
- **Verified by:** headless browser screenshots taken at several timeline points, plus a console-error check.

## Design tokens
- Stage `#000000` · Ink `#F5F5F2` · Dim ink `#8A8F98` · Night Sky `#1E2A44` · Star White `#E8E5DE` · Titanium `#BFC3C9`
- Type: Instrument Sans (Google Fonts), tight tracking, sentence case, with a system-font fallback.
- **Signature idea: "the seam".** The film opens on a single vertical line of light. It turns out to be the folded phone's hinge seen edge-on. At the end, the title unfolds open from that same seam.
- No Apple logo. A small "fan-made concept" note is shown.

## Storyboard (seconds)
1. 0–1.6: the seam of light appears on black.
2. 1.2–4.5: the folded phone is shown edge-on with its hinge spine lit.
3. 4.5–7.5: it rotates to the back (camera plateau, titanium glints).
4. 7.5–10: it rotates to the front and the outer screen wakes.
5. 10–13.5: it unfolds and the inner screen blooms outward from the crease.
6. 13.5–17.5: the title unfolds from the seam, followed by the dates, price and controls.

## Steps
1. Scaffold the HTML, CSS and overlay UI (the Three.js r128 CDN build is the only dependency).
2. Build the geometry: two halves, hinge, cameras, buttons, screens. Add materials and an environment map.
3. Draw the screen textures on canvas.
4. Code the timeline, camera fit and captions.
5. Add the interactive mode: replay, finish, fold, drag. Handle reduced motion.
6. Verify with agent-browser screenshots and console checks, then fix issues.

## Progress
- [x] Steps 1–5 built. Node syntax check passes. The page loads with no console errors or page errors.
- [x] Desktop screenshots at t = 2.6, 6, 9.6, 11.8 and 16.5 all render as storyboarded.
- [x] Fixes from review: less inner-screen glare, less saturated lens rings, hinge spine z-fighting, and the hinge tick visible when open.
- [x] Hinge rework. The partial-cylinder spine left a gap that showed flickering back faces. It's now a solid barrel that scales in for the last 16% of the fold. Checked at t = 9.6, 10.7 and 11.8: clean edge, no poke-through.
- [x] Screen veil. A rough screen material spread the key light into a grey sheen. Fixed with roughness 0.18 and env intensity 0.06. Checked at t = 11.8 and 16.5.
- [x] Controls work: Star White changes the finish (aria-pressed = true), and Fold/Unfold animate with the label flipping.
- [x] Mobile, 390×844: title, captions and controls all fit, with no horizontal overflow.
- [x] Reduced motion: the page skips straight to the end state (interactive, title open).
- [x] No console errors or page errors in any run. The GPU was a Radeon RX 9070 XT with 4× MSAA.
- Known nit: when fully open, a tiny notch shows at the hinge on the top and bottom edges, where the two bevels meet.

## Risks
- ~~The CDN needs an internet connection.~~ Resolved: see "Offline build" below.
- WebGL in headless Chrome may use a software renderer, which is slow but renders.

## Offline build (2026-09-12)
**Task:** make `index.html` run fully offline (including from `file://`) and stay a single file.
**Done =** the page makes zero network requests, renders the same way, and has no console errors.

Steps:
1. Download Three.js r128 from cdnjs and check it against the page's SRI hash. The hash matches.
2. Download the Instrument Sans variable woff2 files (latin + latin-ext, wdth 75–100, wght 400–700).
3. Inline Three.js as a `<script>` block. Replace the Google Fonts `<link>` tags with `@font-face` rules that use base64 `data:` URIs.
4. Update the fatal-error message, which still mentions an internet connection.
5. Verify: grep for `http`, and load the page in agent-browser with the network blocked. Check requests, console, fonts and a screenshot.

Progress:
- [x] Steps 1–4 done. `index.html` grew from 32 KB to 721 KB. It was built by an inliner script in the session scratchpad, which asserts that each replacement matches exactly once.
- [x] No external `src`, `href`, `url()` or `@import` is left. Both inline scripts compile. Both embedded fonts decode as WOFF2.
- [x] Opened from `file://` with every http and https request aborted. The only requests were the `file://` document and `data:` fonts. The console was empty and there were no page errors. `THREE` loaded, Instrument Sans reports `loaded`, and the fatal-error box never appeared.
- [x] Screenshots at t = 11.8 and t = 16.5 match the earlier online renders, including the title and the canvas "9:41" in Instrument Sans.

## GitHub (2026-09-12)
- Repo: `blurboy1985/iphone-duo-intro`. Private, no GitHub Pages (the user's choices). Branch `main`.
- Commits `index.html` and `PLAN.md`.
- Verify: the remote exists and is private, remote `main` matches the local HEAD, and both files are listed on the remote.
- **Superseded 2026-09-12:** the user asked to publish to GitHub Pages. See "Publishing" below.

## Mail on the screens (2026-09-12)
**Task:** show today's Google mail on the phone's own displays, and open a message by clicking it, in three.js.

**Done =** the inbox is drawn on the 3D screens (not over them), a click on a message opens it,
both displays work, and the page still makes no network requests until the viewer asks it to.

### How it works
- The list and the reader are painted into the existing screen canvases, so they arrive as the
  screens' emissive textures. Nothing is overlaid in HTML.
- A click is a `THREE.Raycaster` cast at the three screen meshes. The hit's `uv` maps straight to a
  texture pixel, because `panelGeo` already normalises each panel's UVs (the inner display's two
  halves take `[0, 0, .5, 1]` and `[.5, 0, 1, 1]` of one 2048 × 1465 canvas). Each painter records
  its tappable rows into `scr.hits`, and the pixel is tested against those.
- Panels are `FrontSide`, so a screen facing away is never picked. The inner pair is additionally
  ignored past 50% fold, when the two halves have shut on each other.
- A touch that lands on glass scrolls or taps; a touch anywhere else still turns the phone. Under
  12 texture px of travel counts as a tap, more counts as a scroll, with momentum and a wheel path.

### Layout
- **Inner 7.6"** — a split view that uses the fold: the day's messages down the left half, the open
  message on the right, with the crease as the divider.
- **Outer 5.4"** — the same inbox one pane at a time, with a back chevron. Its hinge-side margin is
  wider (106 px vs 52) because the hinge barrel stands proud of this display and shaves that edge
  when the phone is shut.
- Opening mail pushes the camera in (`view.mailOpenDist` / `mailClosedDist`), stops the idle float
  so type stays sharp and taps land, and fades the title out. Closing it eases back.
- On a portrait viewport a 1.4:1 landscape display cannot be legible, so mail opens folded, on the
  cover display — the way you would really read it. Unfold is still one tap away.

### Data
- Default is a **clearly labelled sample inbox** ("Sample inbox" on screen and in the dock), dated to
  today. This keeps the offline promise: still zero network requests on load.
- **Connect Gmail** is opt-in. It loads Google Identity Services on click, asks for
  `gmail.readonly`, and lists `in:inbox after:<local midnight epoch>`, newest first, up to 15.
  Bodies prefer `text/plain` and fall back to stripped `text/html`.
- The viewer supplies their own OAuth **Web application** client ID (a sheet, or `?gmail_client_id=`).
  Only that ID is stored, in `localStorage`. The access token is held in memory for the tab and is
  revoked on Disconnect. Nothing is sent anywhere but Google.

### Accessibility
- A visually hidden mirror (`#mailsr`) lists the messages as real buttons and carries the open
  message's text, so the inbox is reachable without pointing at a 3D surface. Escape closes a message.

### Verify
- Headless Chromium, `file://`, every http/https request aborted. Requests were the document alone in
  every run; console and page errors clean throughout.
- Raycast picking proved end to end: hover until the canvas cursor turns to a pointer, click, then
  read the opened message back out of the a11y mirror. Works on the inner list and on the cover
  display, and the cover's back chevron closes the message.
- A drag over the list scrolls without opening a different message; a drag off the glass still turns
  the phone; wheel scrolls the pane under the cursor.
- Star White repaints the mail UI in its palette. Replay closes mail and restores the film.
- 390 × 844: opens folded, cover display legible, no horizontal overflow.
- Intro regression at t = 2.6, 6, 9.6, 11.8 and 16.5 — unchanged, console clean.
- `?mail=1` opens straight into the inbox (for review and screenshots).

### Notes
- The sample inbox is fictional and labelled as such; no real mail is committed to this repo.
- `accounts.google.com` and `gmail.googleapis.com` appear only as strings inside the script. The page
  still has no external `src`, `href`, `url()` or `@import`.

## Publishing (2026-09-12)
**Task:** put the page on GitHub Pages.

- `.github/workflows/pages.yml` deploys on every push to `main`, and on demand via `workflow_dispatch`.
- It copies `index.html` alone into `_site`, so `PLAN.md` and the workflow itself are not published.
- `concurrency: pages` with `cancel-in-progress: false` lets a running publish finish.

**Pages has to be switched on by hand, once.** The first two runs both died at
`Create Pages site failed: Resource not accessible by integration`. `actions/configure-pages` was
being asked to enable Pages itself via `enablement: true`, and it cannot: `GITHUB_TOKEN`'s
`pages: write` covers *deploying to* an existing site, but *creating* one needs repo admin, which
`GITHUB_TOKEN` never has. `enablement: true` has been removed, and the switch-on is a one-time
Settings -> Pages -> Build and deployment -> Source: GitHub Actions.

**Repository visibility.** Made public on 2026-09-12. That was needed for Pages on a free plan
(Pages serves private repos only on Pro, Team or Enterprise) but it was *not* the cause of the two
failures above — the same error occurred after the repo went public. Either way the published site
is world-readable; per-site access control is Enterprise Cloud only.

**Note for the Gmail feature.** Pages serves over https, so Google Identity Services will run there,
unlike `file://`. Connecting needs `https://blurboy1985.github.io` added to the OAuth client's
authorised JavaScript origins. The sample inbox needs nothing.

**Live 2026-09-12.** Run 4 of the workflow deployed successfully once Pages was switched on by hand.
Site: https://blurboy1985.github.io/iphone-duo-intro/ — `README.md` carries the link, the Gmail
setup steps and the third-party notices for the inlined Three.js and Instrument Sans.

## Client ID reset (2026-09-12)
**Bug:** `connectGmail()` read the saved client ID and only opened the entry sheet when there was
none, so a mistyped ID was stuck — no way to change it from the UI, and Disconnect kept it too.

**Fix:** a **Change client ID** button in the mail bar, shown whenever an ID is stored; it opens the
sheet with the value selected for overtyping. A **Forget saved ID** button inside the sheet clears
storage. `?gmail_client_id=` now overwrites the stored value, so the URL is a reset as well. The
sign-in failure messages name the button, since a wrong ID and a missing authorised origin both
surface as a generic GIS error.

**Verify:** over http (localStorage needs a real origin) — seed a wrong ID, confirm the button
appears, the sheet prefills and selects it, Forget clears storage and re-hides the button, Connect
then reopens the sheet, submitting saves the new ID, and a URL param replaces a stored one. Full
mail and intro regressions re-run clean.

## Client ID validation (2026-09-12)
**Why:** a bad client ID surfaces only as Google's bare `401: invalid_client` page, which says
nothing about what was actually pasted.

**Fix:** `clientIdProblem()` checks the value before any sign-in is launched, and names the specific
mistake — a `GOCSPX-` client secret, an `AIza` API key, an embedded space, a missing
`.apps.googleusercontent.com` suffix, or a shape that is not
`123456789012-abc.apps.googleusercontent.com`. The message appears inline in the sheet rather than
in the dock. A quoted paste has its surrounding quotes stripped. Advising rotation on a pasted
secret is deliberate: it should not have been typed into a page at all.

**Verify:** five bad inputs each produce their specific message, leave the sheet open and store
nothing; a well-formed id is accepted and stored; a quoted paste is cleaned.

**Field note.** Both failure modes were hit for real, in order: `401: invalid_client` (the ID was not
a real web client), then `400: origin_mismatch` (ID good, origin not registered). They are distinct,
and the second is the better sign. README documents both.

## Origin readout (2026-09-12)
**Why:** `400: origin_mismatch` persisted after the right-looking origin was registered, and nothing
on the page showed which origin it was actually sending. Transcribing it by hand is the weak point.

**Fix:** the connect sheet prints `location.origin` with a Copy button, so the registered value can
be pasted rather than typed. A `file://` page says plainly that sign-in cannot work there, and hides
the copy button. The sign-in failure text now quotes the origin too.

**Verify:** over http the real origin is printed and copyable, and the copy button falls back to
selecting the text when the clipboard API is unavailable rather than throwing; from `file://` the box
flags itself and hides the button.
