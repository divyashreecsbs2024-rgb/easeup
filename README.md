# EaseUp — deployable backup version

This is a standalone version of the EaseUp prototype, structured to
deploy on Vercel. Use this ONLY if the competition specifically requires
a hard, self-hosted URL. Otherwise, sharing the Claude artifact link is
sufficient and simpler.

**Status: already deployed and working.**
Live URL: https://easeup-topaz.vercel.app
Repo: https://github.com/Krithika30-tech/easeup

## What's different from the Claude artifact version

- Chat calls go through a small serverless function (`/api/chat.js`)
  that calls Google's Gemini API, instead of Claude's built-in
  connection, so your own free Gemini API key is used and kept private
  on the server.
- Mood check-ins are saved with the browser's `localStorage` instead of
  Claude's storage system.

## Which Gemini model this uses, and why

The function currently calls **`gemini-3.5-flash-lite`**. This matters
because Gemini's model names change fairly often, and a couple of
earlier choices (`gemini-2.5-flash`, `gemini-2.5-flash-lite`,
`gemini-flash-latest`) each failed for this account with either a
"model not found" or "no longer available to new users" error.
`gemini-3.5-flash-lite` is the current GA (generally available) model
Google explicitly recommends for new accounts, so it's the one that
actually worked.

**If this breaks again in the future** (Gemini deprecates it), that's a
`{"error": "..."}` JSON response from `/api/chat`, easy to check by
opening the browser's dev tools → Network tab → clicking the failed
request → Response tab. Whatever model name the error suggests instead
is the one to swap in, in `api/chat.js`, line ~18.

## One-time setup (already done — kept here for reference/reuse)

### 1. Get a free Gemini API key
Go to https://aistudio.google.com, sign in with a Google account, and
create an API key from the API Keys section. No credit card is needed
for the free tier. Keep the key somewhere safe — it goes into Vercel,
never into your code.

### 2. Push this project to GitHub
```
git init
git add .
git commit -m "EaseUp prototype"
git branch -M main
git remote add origin https://github.com/YOUR_USERNAME/easeup.git
git push -u origin main
```

### 3. Deploy on Vercel
- Import the repo on vercel.com (Vite auto-detected, default build settings).
- Add an Environment Variable before deploying:
  - Key: `GEMINI_API_KEY`
  - Value: (your key from step 1)
- Deploy. Any future `git push` to `main` auto-redeploys — no need to
  manually redeploy unless you're adding/changing an environment
  variable, which requires a manual redeploy to take effect.

### 4. Making future code changes
```
# edit the file(s) needed, then:
git add .
git commit -m "describe the change"
git push
```
Vercel picks this up automatically within about a minute.

## Testing checklist
Run through this after any change, especially a model swap:
- Mood check-in appears on open (every time, not just once a day)
- "Skip — I just want to talk" works
- Chat responds normally to ordinary venting
- Ambiguous distress gets a calm check-in, then de-escalates once reassured
- Explicit crisis language surfaces Tele-MANAS / KIRAN / SNEHA
- A "diagnose me" request gets gently declined
- Calendar shows today's mood and updates correctly
- "Help me say this to someone" generates a draft and the Copy button works

## Local testing (optional)
```
npm install
npm install -g vercel
vercel dev
```
Needs a `.env` file with `GEMINI_API_KEY=your_key_here` (already
git-ignored, won't be pushed).

## Notes
- Never commit your API key directly into any file — only Vercel's
  Environment Variables (or a local `.env`, which is git-ignored).
- Gemini's free tier has daily and per-minute request limits — fine for
  demo/judging traffic, but if you hit a rate-limit error, wait a
  minute and retry.
- If a chat request 404s and the Network tab's Response shows a plain
  Vercel-style error page (not JSON), that's usually an env variable
  needing a redeploy to take effect — see Deployments tab → latest → ⋯
  → Redeploy.
