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
cp config.example.js config.js
```

Open `config.js` and fill in each value:

- **`GEMINI_API_KEY`** — from AI Studio. If set, the app uses this key for everyone and the "Bring your own key" prompt is hidden. If left empty, every visitor must paste their own key.
- **`FIREBASE`** — in the Firebase Console, create a project → add a web app → copy the config object. Then enable **Email/Password** under Authentication → Sign-in method.
- **`EMAILJS_*`** — create an EmailJS service + template (the template must accept `to_email`, `otp`, and `app_name` variables).

### 2. Lock down Firebase

In the Firebase Console:

- **Authentication → Settings → Authorized domains** — add only the domain(s) you'll deploy to. This is the real security boundary; the public `apiKey` is fine to ship in the bundle, but authorized domains stop anyone from reusing your project from a different host.

### 3. Run locally

Open `pokesearch.html` directly in a browser, or serve the folder over HTTP (Firebase Auth requires `http://localhost`, not `file://`):

```sh
npx http-server -p 5500
# then open http://localhost:5500/pokesearch.html
```

### 4. Deploy (pick one — all free)

This is a static site: just `pokesearch.html` + `config.js` + the `config.example.js` template. Don't deploy `node_modules/` (not needed at runtime; Firebase loads via CDN).

**Firebase Hosting** (recommended if you already use Firebase)

```sh
npm install -g firebase-tools
firebase login
firebase init hosting    # public dir: . (current folder), single-page app: No
firebase deploy
```

**Netlify** — drag the folder onto [app.netlify.com/drop](https://app.netlify.com/drop), or connect a GitHub repo. Add `config.js` via the dashboard (Site settings → Build & deploy → Environment, or upload directly) — never commit it.

**Vercel** / **Cloudflare Pages** — same idea: connect the repo, set the output to root, deploy.

**GitHub Pages** — works but requires a public repo on the free plan, and `config.js` would need to be committed (defeating its gitignore). Not recommended.

## File overview

- `pokesearch.html` — entire app (HTML + CSS + JS in one file).
- `config.example.js` — template; commit this.
- `config.js` — your real keys; **gitignored**, never commit.
- `pokesearch-backup.html` — older snapshot.
- `package.json` — only there to pull in the `firebase` npm package for local development; the deployed site loads Firebase from a CDN.

## Notes on cost & security

- **Gemini key** is the only one with real per-call cost. The BYO flow keeps the hosted site free.
- **Firebase `apiKey`** is *meant* to be visible in client code — it identifies your project, not authenticates it. Security comes from auth rules + authorized domains.
- **EmailJS public key** is also meant to be client-visible; quota is per-template.
- localStorage scope: a user's Gemini key lives only in the browser/profile that set it. Clearing site data removes it.
