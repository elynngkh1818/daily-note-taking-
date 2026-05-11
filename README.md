[README.md](https://github.com/user-attachments/files/27583908/README.md)
# The Daily Ledger — PWA

A personal daily journal that installs to your phone, tablet, or desktop like a native app. Notes, highlights, lowlights, todos, tags, and a sticky-note remark corner. All data stays on your device.

## What's in this folder

```
daily-ledger-pwa/
├── index.html              the app
├── manifest.json           PWA install metadata
├── sw.js                   service worker (offline support)
├── icon-192.png            app icon (192×192)
├── icon-512.png            app icon (512×512)
├── icon-maskable-512.png   adaptive Android icon
├── apple-touch-icon.png    iOS home-screen icon
└── favicon-32.png          browser tab icon
```

## Why you need to host it (can't just double-click)

PWAs require HTTPS and a real URL — service workers won't register on `file://`. Pick one option below.

---

## Option 1 — Netlify Drop (easiest, ~30 seconds)

1. Go to **https://app.netlify.com/drop**
2. Drag this entire `daily-ledger-pwa` folder onto the page
3. Netlify gives you a URL like `https://random-name-12345.netlify.app`
4. Open it on your phone or desktop and install (see "Installing" below)

Free, no account needed for a temporary site. Sign up if you want to keep the URL forever.

---

## Option 2 — GitHub Pages (free, permanent URL)

1. Create a new GitHub repo (e.g. `my-daily-ledger`), public.
2. Upload all files from this folder to the repo root.
3. Repo → **Settings** → **Pages** → **Source: Deploy from a branch** → pick `main`, folder `/ (root)` → Save.
4. Wait ~1 minute. Your URL will be `https://YOUR-USERNAME.github.io/my-daily-ledger/`
5. Open it and install.

---

## Option 3 — Run locally for testing

You need a tiny local web server (just opening the file won't work for the service worker).

**With Python (already installed on Mac/Linux):**
```bash
cd daily-ledger-pwa
python3 -m http.server 8080
```
Then open `http://localhost:8080` in a browser.

**With Node:**
```bash
npx serve daily-ledger-pwa
```

Note: PWA install prompts only appear on HTTPS or `localhost`, so local testing works fine.

---

## Installing the app

Once the URL is open in a browser:

- **iPhone / iPad (Safari):** Share button → **Add to Home Screen**
- **Android (Chrome):** Three-dot menu → **Install app** (or it may prompt automatically)
- **Mac / Windows (Chrome / Edge):** Look for the install icon in the address bar (a small monitor with an arrow) → **Install**

After install you'll have a "Ledger" app icon. Tapping it opens the app full-screen with no browser chrome.

## Data & privacy

All entries save to your device's `localStorage`, scoped to the URL the app is installed from. **Different URL = different data.** If you switch hosts later, your old entries won't carry over automatically (they're not synced anywhere — this is a feature, not a bug).

To export: open browser DevTools → Application → Local Storage → copy the keys starting with `entry:`.

## Updating the app

If you change the code, bump the `CACHE` version in `sw.js` (e.g. `'daily-ledger-v1'` → `'daily-ledger-v2'`) so installed users get the new version on next open.

## Offline

After first visit the service worker caches everything (including the React/Tailwind/Babel CDN scripts), so the app works fully offline from then on.
