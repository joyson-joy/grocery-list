# CLAUDE.md

## What this app is

A shared household grocery list PWA named "Gia's Grocery List". Single HTML file, no build step, no framework. Deployed to GitHub Pages via GitHub Actions.

## Tech stack

- **Single file**: all HTML, CSS, and JS live in `index.html` — no separate files, no bundler
- **Storage**: JSONBin.io (`BIN_ID = '6a39f2fbda38895dfeef1169'`) — a simple JSON key/value REST API
- **API key**: injected at deploy time via GitHub Actions secret `JSONBIN_API_KEY`; the placeholder `%%JSONBIN_API_KEY%%` in `index.html` is replaced by `sed` in the deploy workflow (`.github/workflows/deploy.yml`)
- **Deployment**: GitHub Pages, auto-deploys on push to `main`

## Data model

All state lives in one in-memory object `data` which is fetched from JSONBin on load and written back on every change (debounced 600ms):

```js
data = {
  stores: [{ id: string, name: string }],
  items: [{
    id: string,
    name: string,
    qty: string,       // optional, e.g. "2 gallons"
    storeId: string,   // references stores[].id
    purchased: boolean,
    addedAt: number,   // Date.now() timestamp
    purchasedAt: number | null,
  }]
}
```

IDs are `Date.now().toString()` (millisecond timestamp strings).

## Architecture

- **`loadData()`** — GET `/v3/b/{BIN_ID}/latest` from JSONBin, returns `json.record`
- **`saveData()`** — PUT to JSONBin with full `data` object
- **`scheduleSave()`** — debounces `saveData()` by 600ms; called after every mutation
- **`render()`** — re-renders the current tab; called after every state change

## UI structure

3 tabs (sticky tab bar at top 72px):
1. **List** — shopping items grouped by store, filter chips for each store, FAB = "Add Item"
2. **Purchased** — checked-off items grouped by date (Today / Yesterday / weekday), with "Clear all"
3. **Stores** — manage stores, FAB = "Add Store"

FAB is context-sensitive: changes label/action based on active tab; hidden on Purchased tab.

Modals are bottom-sheets (`.modal-overlay` + `.modal`), slide-up animation. Two modals: `item-modal` and `store-modal`.

## CSS conventions

- CSS custom properties defined in `:root` — always use these instead of raw hex values:
  - `--primary: #3b6fd4`, `--accent: #5b8dee`, `--bg: #f0f4ff`, `--surface: #fff`, `--text`, `--text-muted`, `--border`, `--radius: 14px`
- Blue gradient `linear-gradient(135deg, var(--primary), var(--accent))` is used for header, FABs, checkboxes, store avatars — keep this consistent
- Max-width `480px`, centered — mobile-first single-column layout

## Key behaviors

- Checking an item (`toggleItem`) sets `purchased=true`, `purchasedAt=Date.now()`, then re-renders after 180ms delay (allows checkbox animation to finish)
- Deleting a store also deletes all its items and resets the active filter to 'All'
- Store names must be unique (case-insensitive check on add)
- `esc()` helper must be used on all user-generated content rendered into innerHTML
- Filter chips include "All" + one chip per store; active filter persists across renders
- On load: full-screen spinner shown, removed after JSONBin fetch resolves
- Sync bar (top strip) shows "Saving…" / "Saved" / error states

## Deployment notes

- The `%%JSONBIN_API_KEY%%` placeholder must remain as-is in `index.html` — the CI workflow replaces it with `sed`
- Never commit a real API key into the file; it must stay as the placeholder string
- No build command — the file is deployed as-is after secret injection
