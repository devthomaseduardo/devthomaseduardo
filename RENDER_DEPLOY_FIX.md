# Render deploy fix for `MODULE_NOT_FOUND: /opt/render/project/src/bot.js`

Your deploy log shows Render is trying to execute:

```bash
node bot.js
```

and failing with:

```txt
Error: Cannot find module '/opt/render/project/src/bot.js'
```

This means the **Start Command** configured in Render does not match the actual entry file in your repository.

## What to change in Render

Open your Render service settings and update:

- **Build Command**: keep whatever matches your project (`npm ci --omit=dev` if lockfile exists)
- **Start Command**: set to your real app entry point, for example one of:
  - `npm start`
  - `node server.js`
  - `node index.js`
  - `next start`

Do **not** use `node bot.js` unless `bot.js` exists at the repo root.

## Quick local checks before redeploy

Run these inside the project root:

```bash
cat package.json
npm run
```

- If there is a `"start"` script, prefer `npm start` in Render.
- If there is no start script, use the actual server file directly (for example `node server.js`).

## Port requirement on Render

Render expects your process to bind to `process.env.PORT`.
Make sure your server listens like this:

```js
const PORT = process.env.PORT || 3000;
app.listen(PORT, () => console.log(`listening on ${PORT}`));
```

Without this, Render may continue showing "No open ports detected".

## Why this happened

From your timestamps (April 18, 2026), the build completed successfully, then failed only at runtime when starting the process. That pattern usually indicates a **wrong Start Command**, not an install/build issue.
