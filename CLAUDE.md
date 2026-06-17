# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**ProtetoBanda** is a single-file static web application for tracking volunteer reading hours at a community library ("Biblioteca Comunitária"). The UI is entirely in Portuguese. There is no build step, no package manager, and no dependencies to install — open `index.html` directly in a browser.

## Running the App

```bash
# Open directly in browser (no server needed)
open index.html

# Or serve locally to avoid CORS issues with Firebase
python3 -m http.server 8080
# then visit http://localhost:8080
```

There are no tests, no linter, and no CI pipeline.

## Architecture

Everything lives in **`index.html`** (~700 lines). It is divided into three sections:

1. **`<style>` (lines 1–155):** All CSS using custom properties defined on `:root`. No external stylesheet.
2. **`<body>` (lines 156–263):** Static HTML markup for every screen. Screens are toggled visible/hidden via JS — they all exist in the DOM at once.
3. **`<script>` (lines 267–695):** All application logic as vanilla ES6+ JavaScript.

### Screen Navigation

All screens (home, coordinator login, volunteer login, dashboard, profile) are `<div>` elements with `class="screen"`. Navigation is done by calling `showScreen(id)`, which sets `display: none` on all screens and `display: flex` (or `block`) on the target. There is no router.

### Data Model (Firebase Firestore)

The app uses the **Firebase 10.7.0 compat SDK** loaded from CDN. The Firestore database has a single collection `"livros"` with two documents:

- **`volunteers`** — contains an `items` array of volunteer objects:
  ```js
  { id, name, grade, year, totalHours, joinedAt }
  ```
- **`sessions`** — contains an `items` array of session log entries:
  ```js
  { id, volunteerId, volunteerName, date, hours, notes, loggedAt }
  ```

Both documents are read once and kept in sync via `onSnapshot` listeners. All writes replace the entire `items` array (`update({ items: [...] })`).

### State

Global state is held in module-level variables at lines 280–286:

```js
let volunteers = [];
let sessions = [];
let currentTab = "overview";
let currentProfile = null;
let logForm = {};
let membersForm = {};
let unsubVols, unsubSess; // Firestore unsubscribe handles
```

### Key Functions

| Function | Location | Purpose |
|---|---|---|
| `initFirebase()` | ~270 | Initialize Firebase app + Firestore |
| `startListeners()` | ~341 | Attach `onSnapshot` listeners |
| `showScreen(id)` | ~316 | Navigate between screens |
| `renderOverview()` | ~355 | Render the main dashboard overview tab |
| `renderLogHours()` | ~466 | Render the hours-logging tab |
| `renderMembers()` | ~517 | Render the members management tab |
| `showProfile(id)` | ~600 | Show individual volunteer profile |

### Design Tokens

All colors and fonts are CSS custom properties on `:root`. Primary palette:

- `--forest` / `--forest-mid` / `--forest-light` / `--forest-pale`: green accent scale
- `--cream` / `--surface`: warm background tones
- `--ink` / `--muted` / `--faint`: text hierarchy
- `--red` / `--red-pale`: error/destructive actions
- Fonts: Playfair Display (headings), Lora (body), DM Mono (code/numbers) — all from Google Fonts

### Access Control

There is no authentication. "Login" screens are password gates checked client-side:
- Coordinator password: hardcoded in the JS
- Volunteers identify themselves by selecting their name from the volunteers list

### Firebase Config

The Firebase config (API key, project ID, etc.) is hardcoded in the `<script>` block around line 270. The project is `biblioteca-comunitaria-a93f9`. The API key is a browser-safe public key (Firebase security is enforced via Firestore Rules on the Firebase console, not by keeping the key secret).
