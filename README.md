# PokéSearch

Identify Pokémon from an image. Upload, paste a URL, take a webcam shot, or screenshot — the app sends it to Google's Gemini vision model and shows the closest matches with stats, types, and links.

Features: image / URL / camera / paste input, dark mode, search history, favorites, and optional accounts (sign-up with email OTP, sign-in, sync).

## Use the hosted version

1. Open the site.
2. Get a free Gemini API key at [aistudio.google.com/apikey](https://aistudio.google.com/apikey).
3. Click the profile icon → **🔑 Gemini API Key** → paste it. Saved in your browser only.
4. Upload an image and hit Identify.

The hosted site runs on a free tier and has no per-user budget for AI calls, so each visitor brings their own Gemini key. Firebase Auth (accounts) and EmailJS (sign-up OTP) are provided by the operator.

## Self-host your own copy

You'll need free accounts at:

- [Google AI Studio](https://aistudio.google.com/apikey) — Gemini API key
- [Firebase](https://console.firebase.google.com/) — auth / accounts
- [EmailJS](https://www.emailjs.com/) — sign-up OTP email (optional; sign-up will fail without it but sign-in and search still work)

### 1. Clone and configure

```sh
git clone <this-repo>
cd pokemon
cp public_api_config.example.js public_api_config.js
cp private_api_config.example.js private_api_config.js
```

Open the two new files and fill in your values:

**`public_api_config.js`** (deployed; values are designed to be client-visible)
- **`FIREBASE`** — in the Firebase Console, create a project → add a web app → copy the config object. Then enable **Email/Password** under Authentication → Sign-in method.
- **`EMAILJS_*`** — create an EmailJS service + template (the template must accept `to_email`, `otp`, and `app_name` variables).

**`private_api_config.js`** (gitignored AND deploy-ignored)
- **`GEMINI_API_KEY`** — leave empty for BYO (every visitor pastes their own key in the app's settings modal). Only set it for personal local dev — `firebase deploy` won't ship this file regardless.

### 2. Lock down Firebase

In the Firebase Console:

- **Authentication → Settings → Authorized domains** — add only the domain(s) you'll deploy to. This is the real security boundary; the public `apiKey` is fine to ship in the bundle, but authorized domains stop anyone from reusing your project from a different host.

### 3. Run locally

Serve the folder over HTTP (Firebase Auth requires `http://localhost`, not `file://`):

```sh
npx http-server -p 5500
# then open http://localhost:5500/
```

On Windows, if PowerShell blocks the npx script, run `npx.cmd http-server -p 5500` instead, or one-time `Set-ExecutionPolicy -Scope CurrentUser RemoteSigned`.

### 4. Deploy (pick one — all free)

This is a static site: only `index.html` is needed at runtime. `firebase.json` (included) tells Firebase Hosting to ship `index.html` and skip `config.js`, `API_S.env`, `node_modules/`, and other dev-only files.

**Firebase Hosting** (recommended if you already use Firebase)

```sh
npm install -g firebase-tools
firebase login
firebase deploy    # firebase.json already configured
```

The first `firebase deploy` will ask which project to link — pick the one matching your `config.js` Firebase IDs.

**Netlify** — drag the folder onto [app.netlify.com/drop](https://app.netlify.com/drop), or connect a GitHub repo. Add `config.js` via the dashboard (Site settings → Build & deploy → Environment, or upload directly) — never commit it.

**Vercel** / **Cloudflare Pages** — same idea: connect the repo, set the output to root, deploy.

**GitHub Pages** — works but requires a public repo on the free plan, and `config.js` would need to be committed (defeating its gitignore). Not recommended.

## File overview

- `index.html` — entire app (HTML + CSS + JS in one file).
- `firebase.json` — Firebase Hosting config; tells `firebase deploy` what to ship and what to skip.
- `.firebaserc` — Firebase project + hosting targets (production / preview); committed so future deploys skip the picker.
- `public_api_config.js` — Firebase + EmailJS values. **Gitignored** (so forks bring their own), **deployed** (so the live site can sign users in). Contents are designed to be client-visible.
- `private_api_config.js` — Gemini key only. **Gitignored AND deploy-ignored.** Stays on your machine only. Empty value means BYO mode for the deployed site.
- `*_api_config.example.js` — templates for the two configs; commit these.
- `pokesearch-backup.html` — older snapshot.
- `package.json` / `package-lock.json` — only there to pull in the `firebase` npm package for local development; the deployed site loads Firebase from a CDN.

## Notes on cost & security

- **Gemini key** is the only one with real per-call cost. The BYO flow keeps the hosted site free.
- localStorage scope: a user's Gemini key lives only in the browser/profile that set it. Clearing site data removes it.
