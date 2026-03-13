# Admin Panel Design — Businka Portfolio

**Date:** 2026-03-13
**Approach:** GitHub API CMS (static site, no server)
**Style:** Dark theme matching main site

---

## Goals

- Photographer and developer can both edit site content without touching code
- Upload/manage photos in 3 portfolio categories + hero cards
- Edit texts: name, tagline, bio, stats, contacts
- Changes auto-deploy via GitHub Pages (~30–60 sec after save)

---

## Architecture

### Content separation

All editable content extracted from `index.html` to a separate `content.json` file.
`index.html` loads it via `fetch('content.json')` on `DOMContentLoaded`, renders dynamically.

```
businka/
├── index.html                    — reads content.json on load
├── content.json                  — all texts + photo paths
├── admin-[8-char-hash].html      — admin panel (hidden URL)
└── photos/
    ├── hero/
    ├── people/
    ├── dogs/
    └── events/
```

### content.json schema

```json
{
  "name": "Имя Фамилия",
  "tagline": "Фотограф · Москва",
  "bio": "Текст биографии...",
  "stats": [
    { "value": "5+",   "label": "лет опыта" },
    { "value": "200+", "label": "проектов" },
    { "value": "150+", "label": "клиентов" }
  ],
  "contacts": {
    "telegram":  "https://t.me/username",
    "instagram": "https://instagram.com/username",
    "phone":     "+7 999 000 00 00"
  },
  "hero": {
    "photos": ["photos/hero/1.jpg", "photos/hero/2.jpg", "photos/hero/3.jpg"]
  },
  "portfolio": {
    "people": ["photos/people/1.jpg"],
    "dogs":   ["photos/dogs/1.jpg"],
    "events": ["photos/events/1.jpg"]
  }
}
```

### index.html changes

- Remove hardcoded `photos` object and all text strings
- Wrap JS initialisation in `fetch('content.json').then(data => { ... })`
- Fill DOM after fetch: name, tagline, bio, stats, contacts, hero cards, masonry grids
- Start hero entrance animation + Intersection Observer only after content is rendered
- Add `<noscript>` fallback message

---

## Admin Panel

### Security

- **Hidden URL:** `admin-[random 8-char hex].html` — not linked anywhere on the main site
- **Password:** checked against SHA-256 hash stored in the file (not plaintext)
- **GitHub token:** stored in `localStorage` after first entry; never embedded in code
- **Token scope:** fine-grained personal access token, `Contents: Read and Write` for this repo only

### Auth flow

1. Open `admin-[hash].html`
2. Enter password → SHA-256 verified client-side
3. If no token in `localStorage` → show token input field
4. Token saved to `localStorage` → load `content.json` from GitHub API

### UI: Text tab

Fields: Name, Tagline, Bio (textarea), 3 stat rows (value + label), Telegram, Instagram, Phone.
All pre-filled from current `content.json` on load.

### UI: Portfolio tab

Sub-tabs: Люди / Собаки / Мероприятия / Hero
Per sub-tab:
- Grid of current photos with `[✕]` delete button on each
- `[+ Добавить фото]` button → file picker

### Save button

Single `[💾 Сохранить]` button in the header.
Commits updated `content.json` to GitHub via API.
Photo uploads happen immediately on file selection (individual commits).

---

## GitHub API Flow

### Load

```
GET /repos/AMNovoselov/businka/contents/content.json
→ decode base64 content
→ store file SHA (required for update)
```

### Photo upload

```
FileReader → base64
→ Canvas resize (max 1200px wide, ~300–500 KB output)
→ PUT /repos/.../contents/photos/{category}/{filename}
   body: { message, content: base64, sha?: existing_sha }
→ add returned path to in-memory portfolio list
```

### Save content

```
PUT /repos/.../contents/content.json
body: {
  message: "content: update via admin panel",
  content: base64(JSON.stringify(data, null, 2)),
  sha: current_file_sha
}
```

GitHub Pages rebuilds automatically in ~30–60 seconds.

---

## Constraints

- Photo files compressed client-side to ≤ 500 KB before upload
- Concurrent edits from two devices may cause SHA conflict — not a concern for single user
- All code in single `admin-[hash].html` file, no external dependencies

---

## Success Criteria

- Photographer can open admin URL, log in, change a contact link, hit Save — site updates in ~1 min
- Developer can upload 10 new photos to a category in one session
- No git knowledge required to operate the admin
