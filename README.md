# Loop — synced habit tracker (PWA)

A habit / task / goal tracker that installs to your Home Screen and **syncs across your iPhone, iPad, Mac, and Windows PC through your own Google Drive** — free, no server, no monthly cost. Your data lives in a hidden per-app folder in *your* Drive.

> **Why not a double-click file anymore?** Cloud sign-in (OAuth) and offline install (service workers) both require an `https://` web address. So Loop now lives at a URL instead of a local file. Hosting it is free (below).
>
> **Apple Watch:** not included on this route — the Watch can only sync through Apple's own iCloud, which can't reach your Windows PC. (That was the tradeoff you picked.)

---

## What's in this folder

| File | What it is |
|------|------------|
| `index.html` | the whole app |
| `config.js` | **the one file you edit** — paste your Google client id here |
| `manifest.webmanifest` | makes it installable |
| `service-worker.js` | offline support |
| `icon-*.png`, `apple-touch-icon.png` | app icons |

---

## Setup (about 15 minutes, one time)

### 1. Put the files online (free, gives you an https URL)

**Option A — GitHub Pages (recommended; stable URL):**
1. Create a new repo, e.g. `loop`, and upload everything in this folder to its root.
2. Repo **Settings → Pages → Build and deployment → Source: "Deploy from a branch"**, branch `main`, folder `/ (root)`, **Save**.
3. After ~1 min your app is at `https://YOURNAME.github.io/loop/`. Keep this URL.

**Option B — Netlify (no git):** go to **app.netlify.com/drop**, drag this folder in, and claim the site so the URL stays fixed. Use the `https://...netlify.app` URL it gives you.

### 2. Create a free Google OAuth client

1. Go to **console.cloud.google.com** → create a project (any name).
2. **APIs & Services → Library →** search **"Google Drive API" → Enable**.
3. **APIs & Services → OAuth consent screen:** User type **External** → fill app name + your email. Under **Test users**, add your own Google account. (Leave it in "Testing" — that's fine for personal use.)
4. **APIs & Services → Credentials → Create credentials → OAuth client ID:**
   - Application type: **Web application**
   - **Authorized JavaScript origins → Add URI:** your site's *origin only, no path*:
     - GitHub Pages: `https://YOURNAME.github.io`
     - (optional, for local testing) `http://localhost:8000`
   - **Create**, then copy the **Client ID** (ends in `.apps.googleusercontent.com`).
   - *No redirect URI needed.*

### 3. Paste the client id

Open `config.js`, put the id between the quotes:
```js
window.LOOP_CONFIG = {
  googleClientId: "1234567890-abcd....apps.googleusercontent.com"
};
```
Save, then re-upload `config.js` (GitHub: commit it; Netlify: re-drag the folder).

### 4. Install on each device + connect once

On each device, open your URL, then install:
- **iPhone / iPad (Safari):** Share → **Add to Home Screen**.
- **Mac (Safari):** File → **Add to Dock**. *(or Chrome: ⋮ → Cast, Save & Share → Install)*
- **Windows PC (Edge/Chrome):** address-bar **install** icon → Install.

Then open Loop → **More → Connect Google Drive** → sign in. First sign-in shows *"Google hasn't verified this app"* → **Advanced → Go to (your app)** — expected, because it's your private app. Do this once per device and they'll all share the same data.

---

## Local testing (optional)
```bash
cd loop-pwa
python3 -m http.server 8000
# open http://localhost:8000  (add that origin in step 2 to allow sign-in)
```

## Good to know
- **Sync model:** the most recently saved device wins. Fine for one person on several devices; if you edit two devices while *both* are offline, the last one to sync overwrites. Open the app (it pulls on launch/focus) before editing on a different device.
- **Reconnecting:** Google's browser-only sign-in gives short-lived access (no server to hold a long-lived token), so every so often you may need to tap **Connect** again. Your data is safe locally in the meantime.
- **Where's my data?** In Drive's hidden *application data folder* — you won't see a `loop-state.json` cluttering your normal Drive. **More → Export JSON** still gives you a manual backup anytime.
- **Updating the app later:** change files, then bump `const CACHE = "loop-v1"` to `"loop-v2"` in `service-worker.js` so devices pull the new version.
- **Rename "Loop":** edit the brand text in `index.html` (and `AVATAR`), plus `name`/`short_name` in `manifest.webmanifest`.

### Prefer OneDrive instead of Google Drive?
The whole sync layer is a single module in `index.html` (the `Cloud = (function(){…})()` block). Swapping it to OneDrive (Microsoft Graph) is a contained change — just say the word and I'll hand you a drop-in replacement plus the Azure app-registration steps.
