# Trip Planner – Agent Instructions

This document explains how to add or update a project in the trip planner web app hosted at:
**https://lironmula.github.io/trip_planner_lironm/**

---

## Repository Structure

```
trip_planner_lironm/
├── index.html          ← the web app (do not modify unless updating the app itself)
├── INSTRUCTIONS.md     ← this file
└── projects/
    ├── 2026_summer_trip.json
    └── <your_new_project>.json
```

---

## How to Add a New Project

### Step 1 – Create the JSON file

Create a file named `projects/<project_name>.json`.

**Naming rules:**
- Use underscores, not spaces: `2027_winter_trip.json`
- Lowercase preferred
- No special characters

### Step 2 – JSON structure

```json
{
  "name": "2027_winter_trip",
  "created": "2027-01-01T00:00:00.000Z",
  "saved":   "2027-01-01T00:00:00.000Z",
  "chats": {
    "p7": [],
    "p9": []
  },
  "notes": {}
}
```

**Fields:**
| Field | Type | Description |
|-------|------|-------------|
| `name` | string | Must match the filename (without `.json`) |
| `created` | ISO 8601 datetime | Creation date |
| `saved` | ISO 8601 datetime | Last save date – update on every write |
| `chats` | object | Chat messages per alternative. Keys must match alternative keys in `index.html` (`p7`, `p9`, etc.) |
| `notes` | object | Free key-value notes. Currently unused by the UI but preserved on save |

### Step 3 – Chat message format

Each item in `chats.p7` or `chats.p9` must follow this structure:

```json
{
  "plan_key": "p7",
  "username": "Liron",
  "message": "This looks great - I prefer the 7-night option",
  "created_at": "2026-06-15T14:30:00.000Z"
}
```

**Validation rules enforced by the UI:**
- `username`: letters only (Hebrew, Latin, Arabic), max 20 characters
- `message`: letters, digits, dashes and spaces only. Apostrophes are removed. All other characters become underscores. Max 1000 characters
- Max **20 messages** per alternative (`p7` or `p9`). Messages beyond 20 are ignored

### Step 4 – Push to GitHub via API

Use the GitHub Contents API. Replace `<TOKEN>` with a Fine-grained personal access token
with **Contents: Read and Write** permission on this repository only.

```bash
# Encode the JSON file
CONTENT=$(base64 -w 0 projects/my_new_project.json)

# Push (new file – no sha needed)
curl -X PUT \
  "https://api.github.com/repos/LironMula/trip_planner_lironm/contents/projects/my_new_project.json" \
  -H "Authorization: token <TOKEN>" \
  -H "Accept: application/vnd.github.v3+json" \
  -H "Content-Type: application/json" \
  -d "{\"message\": \"Add my_new_project\", \"content\": \"$CONTENT\"}"
```

**To update an existing file** you must include the current `sha`:

```bash
# Get current sha
SHA=$(curl -s \
  -H "Authorization: token <TOKEN>" \
  "https://api.github.com/repos/LironMula/trip_planner_lironm/contents/projects/my_project.json" \
  | python3 -c "import sys,json; print(json.load(sys.stdin)['sha'])")

# Update
CONTENT=$(base64 -w 0 projects/my_project.json)
curl -X PUT \
  "https://api.github.com/repos/LironMula/trip_planner_lironm/contents/projects/my_project.json" \
  -H "Authorization: token <TOKEN>" \
  -H "Accept: application/vnd.github.v3+json" \
  -H "Content-Type: application/json" \
  -d "{\"message\": \"Update my_project\", \"content\": \"$CONTENT\", \"sha\": \"$SHA\"}"
```

---

## How the Web App Loads Projects

The app reads project files directly from the **public GitHub raw URL** – no token required for reading:

```
https://raw.githubusercontent.com/LironMula/trip_planner_lironm/main/projects/<name>.json
```

The project list is fetched from the GitHub API:

```
https://api.github.com/repos/LironMula/trip_planner_lironm/contents/projects
```

This lists all `.json` files. The app strips `.json` from each filename to get the project name.

---

## How the App Displays Alternatives

The web app currently has **two hardcoded alternatives** (`p7` and `p9`) defined in `index.html`.

The `chats` object in the JSON file maps to these alternatives by key.

If you need to add a **new alternative**, you must also update `index.html`:
1. Add a new entry to the `PLANS` object (rows, detail, map points)
2. Add a new sidebar button and panel in the HTML
3. Add the new key to the `chats` object in any existing project JSON files

---

## Minimal Valid Project (copy-paste ready)

```json
{
  "name": "new_project_name",
  "created": "2026-01-01T00:00:00.000Z",
  "saved": "2026-01-01T00:00:00.000Z",
  "chats": {
    "p7": [],
    "p9": []
  },
  "notes": {}
}
```

---

## Token Security

- Always use a **Fine-grained token** scoped to this repository only
- Set expiry to 90 days or less
- Never commit the token to the repository
- The token is only needed for **writing** – reading is public and requires no authentication
