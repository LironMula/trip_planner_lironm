# Trip Planner – Agent Instructions

This document is the **single source of truth** for any AI agent adding or updating a project.
App URL: **https://lironmula.github.io/trip_planner_lironm/**

> **Your job is to research the trip and build great content.**
> All technical implementation details are in this file — the prompt you received should contain only trip-specific questions.

---

## Repository Structure

```
trip_planner_lironm/
├── index.html          ← the web app (modify only for new alternatives or map data)
├── INSTRUCTIONS.md     ← this file
└── projects/
    ├── 2026_summer_trip.json
    └── <your_new_project>.json
```

---

## Overview: What Files You Need to Create or Update

| Task | File(s) to touch |
|------|-----------------|
| Add a new project | `projects/<name>.json` only |
| Add/update Q&A or chat | `projects/<name>.json` only |
| Add a new alternative (sidebar tab) | `index.html` — `PLANS`, `MAP_DATA`, HTML panels |
| Update map points for an existing alternative | `index.html` — `MAP_DATA` only |

---

## Part 1 — Project JSON

### Naming rules
- Filename: `projects/<name>.json`
- Name: letters, digits, underscores, hyphens only. No spaces. Max 50 chars.
- Example: `portugal_2027.json`

### Full JSON structure

```json
{
  "name": "portugal_2027",
  "created": "2027-01-01T00:00:00.000Z",
  "saved":   "2027-01-01T00:00:00.000Z",
  "chosen":  null,
  "chats": {
    "p7": [],
    "p9": []
  },
  "notes": {}
}
```

| Field | Type | Notes |
|-------|------|-------|
| `name` | string | Must match filename (without `.json`) |
| `created` | ISO 8601 | Creation timestamp |
| `saved` | ISO 8601 | Update on every write |
| `chosen` | `"p7"` / `"p9"` / `null` | Which alternative opens first |
| `chats` | object | Keys must match alternative keys in `index.html` |
| `notes` | object | Reserved — leave as `{}` |

### Chat message format

```json
{
  "plan_key":   "p7",
  "username":   "Liron",
  "message":    "This looks great",
  "created_at": "2026-06-15T14:30:00.000Z"
}
```

**Security rules (enforced by UI):**
- `username`: letters only (Hebrew/Latin/Arabic), max 20 chars
- `message`: letters, digits, dashes, spaces only — no `< > " ' / = \`` — max 1000 chars
- Max **20 messages** per alternative

---

## Part 2 — Alternatives in index.html

The app has **two hardcoded alternatives**: `p7` and `p9`.
Each is defined in three places inside `index.html`:

### 2a. `PLANS` object — trip data

Located inside `<script>` near the top. Structure:

```javascript
const PLANS = {
  p7: {
    rows: [
      // One entry per day — shown in the summary table
      { d:'13.8', r:'Airport → Lake Como', dr:'1h00m', s:'Menaggio', z:'zi' },
      // z values: 'zi'=Italy, 'zk'=Konstanz/Switzerland, 'zf'=forest, 'zx'=flight day
    ],
    det: [
      // One entry per day — shown in the detailed plan
      {
        d: '13.8',
        t: 'Arrival at Lake Como',
        z: 'zi',
        m: 'Morning: ...',   // בוקר
        n: 'Noon: ...',      // צהריים
        e: 'Evening: ...',   // ערב
        sp: 'Split plan for special needs child: ...'
      },
    ],
    qa: [
      // Q&A summary — ordered from GENERAL to SPECIFIC
      // First: what is the destination / why go there
      // Then: logistics (car, cost, borders, food restrictions)
      // Then: activities, accommodation
      // Last: fine-grained / personal specifics
      { q: 'What is the Black Forest?', a: 'The Black Forest is...' },
      { q: 'Do we need a rental car?',  a: 'Yes, because...' },
      // ... more Q&A
    ]
  },
  p9: {
    // same structure
  }
};
```

**Important — apostrophes in Hebrew text:**
Hebrew words containing `'` (e.g. בלאג'יו, צ'ק אין) **must** be escaped as `\'` inside single-quoted JS strings, or the entire script will silently break. Use `\'` or switch to backtick strings.

### 2b. `MAP_DATA` object — map visuals

```javascript
const MAP_DATA = {
  p7: {
    sleeps: [
      // Overnight locations — shown as large numbered circles, connected by solid line
      { lat: 45.62, lng: 8.73, label: 'Malpensa',  nights: null,  color: '#888888' },
      { lat: 46.02, lng: 9.24, label: 'Menaggio',  nights: [1,2], color: '#c8773a' },
      // nights: [firstNight, lastNight] or null for airports/transit
      // label must be unique — used to link attractions
    ],
    attractions: [
      // Day-trip locations — shown as small numbered circles
      // connected by dashed colored line to their base sleep location
      { lat: 45.98, lng: 9.26, label: 'Bellagio', day: '14.8', base: 'Menaggio' },
      // base: must exactly match a label in sleeps[]
      // color is inherited automatically from the base sleep
    ]
  },
  p9: { sleeps: [...], attractions: [...] }
};
```

**Visual result:**
- Sleep markers: large circle showing `1–3 / לילות`, connected by solid dark route line
- Attraction markers: small numbered circles (1, 2, 3...), colored to match their base
- Spoke lines: dashed colored lines from each sleep to its attractions

### 2c. HTML panels — sidebar and content

Each alternative needs:
1. A sidebar button (inside `.alts-sidebar`):
```html
<button class="alt-btn" id="sidebtn-p7" onclick="switchAlt('p7',this)">
  Name<br><span style="font-size:.72rem;font-weight:400;color:var(--rock)">dates · N nights</span>
</button>
```

2. A full content panel (inside `.alts-main`):
```html
<div class="alt-panel active" id="panel-p7">
  <div class="alt-card">
    <div class="alt-hdr">
      <div>
        <div class="alt-title">
          <span class="chosen-badge" id="chosen-badge-p7">✓ מועדף</span>
          Alternative Name
        </div>
        <div class="alt-meta">dates · N nights</div>
      </div>
      <button class="choose-btn" id="choose-btn-p7" onclick="setChosen('p7')">קבע כמועדף</button>
    </div>
    <div class="alt-body">
      <div class="qa-section" id="qa-p7"></div>
      <!-- version bars, table, map, detail, chat — copy from existing panel -->
    </div>
  </div>
</div>
```

---

## Part 3 — Pushing Files to GitHub

Use the GitHub Contents API with a Fine-grained token
(Contents: Read and Write, scoped to `trip_planner_lironm` only).

### Push a new file
```python
import urllib.request, json, base64

TOKEN = "<YOUR_TOKEN>"
API   = "https://api.github.com/repos/LironMula/trip_planner_lironm/contents/projects/my_project.json"
HDRS  = {"Authorization": f"token {TOKEN}",
         "Accept": "application/vnd.github.v3+json",
         "Content-Type": "application/json"}

with open("my_project.json", "rb") as f:
    content = base64.b64encode(f.read()).decode()

payload = json.dumps({"message": "Add my_project", "content": content}).encode()
urllib.request.urlopen(urllib.request.Request(API, data=payload, headers=HDRS, method="PUT"))
```

### Update an existing file (requires sha)
```python
# 1. Get current sha
with urllib.request.urlopen(urllib.request.Request(API, headers=HDRS)) as r:
    sha = json.load(r)["sha"]

# 2. Push with sha
with open("my_project.json", "rb") as f:
    content = base64.b64encode(f.read()).decode()

payload = json.dumps({"message": "Update", "content": content, "sha": sha}).encode()
urllib.request.urlopen(urllib.request.Request(API, data=payload, headers=HDRS, method="PUT"))
```

Same pattern for `index.html` — just change the API path to `.../contents/index.html`.

### Reading (no token needed — repo is public)
```
https://raw.githubusercontent.com/LironMula/trip_planner_lironm/main/projects/<name>.json
```

---

## Part 4 — Security Rules

- Never inject raw user text into `innerHTML` — always escape `< > " ' / = \``
- Project names: letters, digits, underscores, hyphens only (validated by UI)
- Chat messages loaded from GitHub are re-sanitized before display
- Token: Fine-grained, repo-scoped, 90-day expiry max — never commit to repo

---

## Part 5 — Minimal Copy-Paste Templates

### Minimal project JSON
```json
{
  "name": "my_trip",
  "created": "2026-01-01T00:00:00.000Z",
  "saved":   "2026-01-01T00:00:00.000Z",
  "chosen":  null,
  "chats":   { "p7": [], "p9": [] },
  "notes":   {}
}
```

### Q&A ordering (always general → specific)
1. What is the destination / why go there
2. Is a rental car needed
3. How many nights / overall route
4. Estimated costs
5. Border crossings / logistics
6. Food restrictions / dietary needs
7. Activities for the specific travelers
8. Which attractions are covered / not covered
9. Fine-grained personal specifics
