# iPhone Duo — 3D intro

A cinematic, real-time 3D intro for a foldable iPhone concept, with a working
mail app rendered **on** the phone's own screens.

**[Open the live page →](https://blurboy1985.github.io/iphone-duo-intro/)**

> **Fan-made concept. Not affiliated with, endorsed by, or connected to Apple.**
> The device, its specifications and the launch dates are speculative, drawn from
> public reporting. No Apple trademarks or logos are used.

One file, no build step, no dependencies to install. `index.html` is fully
self-contained — Three.js and the webfont are inlined — so it runs from a local
copy with **zero network requests**.

---

## The intro

A ~17-second film. It opens on a single vertical line of light, which turns out
to be the folded phone's hinge seen edge-on; at the end the title unfolds open
from that same seam.

Then it hands over to you: drag to turn the phone, fold and unfold it, switch
between the Night Sky and Star White finishes, or replay.

`prefers-reduced-motion` skips straight to the interactive end state.

## Mail on the screens

Press **Mail** in the dock. Today's inbox is painted into the screen canvases,
so it arrives as the displays' emissive textures rather than as an HTML panel
floating over the render — and a click on a message is a `THREE.Raycaster` cast
at the screen meshes, with the hit's UV mapped back to a texture pixel.

- **7.6-inch inner display** — a split view that uses the fold: the day's
  messages down the left half, the open message on the right, crease as divider.
- **5.4-inch cover display** — the same inbox one pane at a time, with a back
  chevron. On a narrow screen mail opens folded, the way you would really read it.
- Touch the glass to scroll or tap; touch anywhere else to turn the phone.

The inbox is a clearly labelled **sample** until you connect Gmail, which is how
the page keeps its no-network-requests promise.

## Using your real Gmail

You supply your own Google OAuth client. It's free and stays in Testing mode, so
no Google verification review is needed.

1. **Enable the API.** [Google Cloud Console](https://console.cloud.google.com) →
   a project → **APIs & Services → Library** → **Gmail API** → **Enable**.
2. **Consent screen.** **APIs & Services → OAuth consent screen** → User type
   **External** → fill in the app name and contact emails → under **Scopes** add
   `https://www.googleapis.com/auth/gmail.readonly` → under **Test users** add
   your own address. Leave the app in **Testing**.
3. **Credentials.** **APIs & Services → Credentials → Create Credentials →
   OAuth client ID** → Application type **Web application** → under
   **Authorized JavaScript origins** add:

   ```
   https://blurboy1985.github.io
   ```

   The **origin only** — no path. A mismatch here is the most common failure.
   No redirect URI is needed; the page uses the popup-based GIS token flow.
4. **Connect.** Open the page → **Mail** → **Connect Gmail** → paste the client
   ID → pick your account → grant read-only access.

Google will warn that the app is unverified. That is expected for your own
client: **Advanced → Go to … (unsafe)**.

### Two errors you may hit, and what each one means

**`Error 401: invalid_client`** — Google has no client with that ID. The ID itself is
wrong: a truncated paste, or the client secret, the API key or the project
number copied instead. It is not an origin problem; the request never got that
far. The page now checks the shape before sending you to Google and says which
of these it looks like.

**`Error 400: origin_mismatch`** — the ID is good and Google found it, but the
address you are viewing from is not registered on that client. Fix it in
**Credentials → your OAuth 2.0 Client ID → Authorized JavaScript origins**:

```
https://blurboy1985.github.io
```

The **Connect Gmail** box prints this page's own origin with a Copy button, so
paste that rather than typing it — it is exactly the string Google will be sent.

Three things go wrong here:

- It must go under **Authorized JavaScript origins**, *not* Authorized redirect
  URIs. This flow never uses a redirect URI.
- **Origin only** — scheme and host. No `/iphone-duo-intro/`, no trailing slash.
  The path is not part of an origin.
- Google can take a few minutes to propagate the change. If it still fails
  straight after saving, wait and retry before changing anything else.

To run it from a local server as well, add that origin too, e.g.
`http://localhost:8000`.

If it still fails after adding the origin, check in this order:

1. **Wait.** Google's own note says a change can take five minutes to a few hours.
2. **Same client?** Open the client in Google Cloud and compare its Client ID,
   character for character, against the one in **Change client ID**. Adding the
   origin to a different client than the page is using looks identical from here.
3. **Right box?** Origins and redirect URIs sit next to each other. Only
   **Authorized JavaScript origins** matters for this flow.
4. **Same origin?** Compare what the box prints against what is registered. A
   local copy, a preview deployment or a custom domain is a different origin.

### Pasted the wrong client ID?

The ID is remembered, so there are three ways to change it:

- **Change client ID** in the mail bar — appears as soon as an ID is saved. It
  opens the box with the current value selected, ready to be typed over.
- **Forget saved ID** inside that box clears it entirely.
- Loading the page with `?gmail_client_id=…` replaces whatever is saved.

You will see today's inbox from local midnight, up to 15 messages, newest first.

### What is stored

| | |
|---|---|
| Access token | **In memory only.** Gone on refresh, so you reconnect each visit. Revoked on **Disconnect**. |
| Client ID | `localStorage`, so you only paste it once. Not a secret for web OAuth clients. |
| Your mail | Never stored, never sent anywhere but Google. Nothing leaves your browser. |

Gmail cannot be connected from a local `file://` copy — Google Identity Services
refuses to run on that origin. The sample inbox works everywhere.

## URL parameters

| Parameter | Effect |
|---|---|
| `?mail=1` | Skip the intro and open straight into the inbox |
| `?gmail_client_id=…` | Pre-fill the OAuth client ID |
| `?t=9` | Jump the intro to 9 seconds |
| `?freeze` | Hold that frame (for screenshots and review) |

## Accessibility

The inbox is mirrored into a visually hidden list of real buttons, so it works
without pointing at a 3D surface. Captions are announced, controls are labelled,
and <kbd>Esc</kbd> closes an open message.

## Third-party

Both are embedded in `index.html` rather than fetched, with their notices intact.

- **[Three.js](https://threejs.org) r128** — MIT License, © 2010-2021 three.js authors
- **[Instrument Sans](https://fonts.google.com/specimen/Instrument+Sans)** — SIL Open Font License 1.1

`PLAN.md` has the build log: the storyboard, the geometry and timeline decisions,
and what was verified.
