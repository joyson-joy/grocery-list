# Gia's Grocery List

A shared household grocery list web app. Items are grouped by store, synced via JSONBin, and deployed to GitHub Pages.

## Features

- Add grocery items with optional quantity, assigned to a store
- Drag and drop to reorder items or move them between stores
- Drag and drop to reorder stores
- Purchased items tracked with timestamps, grouped by date
- Refresh button to pull the latest changes made by other household members
- Shared cloud sync via JSONBin (no login required)

## Running locally

**1. Get your JSONBin API key**

Log in to [jsonbin.io](https://jsonbin.io) → go to **API Keys** → copy your Master Key.

**2. Create `dev-config.js`** (already gitignored)

```js
// Local dev only — DO NOT commit (gitignored)
window.DEV_JSONBIN_API_KEY = 'your-jsonbin-master-key-here';
```

**3. Start a local server**

```bash
npx serve -p 3456 .
```

Then open [http://localhost:3456](http://localhost:3456).

> The app reads from `dev-config.js` locally. In production, GitHub Actions injects the real key via the `JSONBIN_API_KEY` repository secret — `dev-config.js` is never deployed.

## Deployment

Pushes to `main` auto-deploy to GitHub Pages via `.github/workflows/deploy.yml`.

Make sure the `JSONBIN_API_KEY` secret is set in your GitHub repository settings under **Settings → Secrets and variables → Actions**.

## Project structure

```
index.html       # entire app — HTML, CSS, and JS in one file
dev-config.js    # local API key (gitignored, never committed)
.gitignore
.github/
  workflows/
    deploy.yml   # injects API key and deploys to GitHub Pages
```
