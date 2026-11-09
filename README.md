# The Lost Memories — A Private Memory Archive

A cinematic treasure hunt that ends in a private antique library. The archive belongs to **Krishna, Hayson, Nilesh and Lizbeth**, with a fifth **Library Book** for shared and unassigned frames.

All media comes from the shared Google Drive folder **“Time in Goa”** (`1ndV0jl5OgWObjlv38V0SacekOz23aQhu`). No stock photographs or generated people are used anywhere.

- A five-scene map journey, then a sealed wooden library door
- One library word, then one password per personal volume
- Five leather-bound books on a walnut shelf
- Every image and film is served by the Node server **after** a session check; direct URLs are refused (401/403)
- **435 Drive files verified** against the local derivatives: 373 photographs and 62 films across the five volumes
- Addressable pages: `/`, `/library` and `/books/:slug`, with browser Back/Forward and deep links
- A private **Upload Treasure** desk: pending queue, duplicate refusal, preview, explicit publish and discard
- Optional, cached visual descriptions under photographs; editable from the viewer. AI captioning is offline in this zero-budget build and no caption is invented

---

## 1. Run it locally

Requirements: **Node.js 20.11+**. ImageMagick is only needed when re-ingesting media.

```bash
cd four-of-us
npm install

# 1. set the five passwords (library + four personal volumes)
npm run passwords
#    or generate strong temporary ones:
npm run passwords -- --generate

# 2. start the archive
npm start
```

Open **http://127.0.0.1:4310**.

`npm run passwords` hashes with scrypt and writes `config/archive.config.json`. Plaintext passwords are never stored. The generated mode prints a password for the library and one per personal volume once — copy them, then rotate with `npm run passwords` at any time (the hashes are read on the next unlock attempt, no restart needed).

Environment variables:

| Variable | Default | Meaning |
| --- | --- | --- |
| `PORT` | `4310` | HTTP port |
| `HOST` | `127.0.0.1` | Bind address. Use `0.0.0.0` behind a deployment platform |
| `OPENAI_API_KEY` | — | Optional; only for `npm run describe` |
| `OPENAI_BASE_URL` | `https://api.openai.com/v1` | Optional OpenAI-compatible vision endpoint |
| `OPENAI_VISION_MODEL` | `gpt-4o-mini` | Optional vision model for descriptions |

---

## 2. What is in the box

```
four-of-us/
├── server/
│   ├── index.js          Express app: sessions, auth routes, protected media, descriptions
│   ├── catalog.js        Joins data/archive.json with data/books.json
│   ├── config.js         Reads config/archive.config.json
│   ├── passwords.js      scrypt hashing + verification
│   ├── sessions.js       Server-side sessions, opaque HttpOnly cookie
│   └── ratelimit.js      Per-IP password attempt limiter
├── scripts/
│   ├── fetch-media.mjs   Drive ingest → thumbnails, display JPEGs, web MP4s
│   ├── verify-media.mjs  Re-lists Drive, compares IDs/files and writes the verification report
│   ├── describe-media.mjs Resumable visual-description CLI
│   └── set-passwords.mjs Interactive password CLI
├── data/
│   ├── books.json        Five volumes: names, numerals, leather, epigraphs
│   ├── experience.json   Journey copy, gate copy and the four clue inscriptions
│   ├── descriptions.json Cached visual descriptions, keyed "slug/key"
│   ├── captions.json     Optional handwritten caption overrides
│   ├── archive.json      Generated media manifest (do not hand-edit)
│   ├── source-listing.json Generated mapping of Drive file → book
│   ├── media-verification.md  Last Drive ↔ local audit
│   └── uploads.json      Private pending-upload queue (gitignored)
├── config/
│   ├── archive.config.example.json
│   └── archive.config.json          Generated, gitignored (hashes only)
├── media/
│   └── books/<book>/     Web derivatives: display + thumb + poster + web video
├── public/
│   ├── index.html
│   ├── css/              base, treasure, landing, library, album
│   ├── js/               app, treasure, journal, album, upload, api, util
│   └── assets/fonts/     Self-hosted Cormorant, EB Garamond, La Belle Aurore
├── screenshots/          Verification captures
├── README.md
├── uploads/pending/       Private raw uploads awaiting publish (gitignored)
└── DEPLOYMENT.md
```

---

## 3. The experience

### Pages and routes

The archive is one server-rendered shell with three real, addressable views. Navigation uses the History API, so Back, Forward, refresh and a pasted link all land on the same place:

| Route | View |
| --- | --- |
| `/` | The journey, gate, or library depending on the session and intro state |
| `/library` | The private bookshelf and the archive desk |
| `/books/:slug` | One volume's story page; the album opens over it |

The server returns `index.html` for these routes and the client resolves the slug. It does not render five separate HTML documents, so shared chrome, auth state and the album are not duplicated. A signed-out deep link stops at the gate; a sealed volume stops at its password card; an unknown slug returns to the library with a notice.

### The journey

`public/index.html` contains one fixed, scrollable journey section (about 5.8 viewports tall). Native scrolling drives a sticky viewport; nothing is hijacked. Five chapters:

1. **The map** — parchment, mountain, forest, river, roads, a dotted route that draws itself and a lit X.
2. **The journey** — the camera pushes across the map; bridge, river and forest move at different depths.
3. **The discovery** — an abandoned mansion emerges and the camera approaches its door.
4. **The treasure** — a brass key and a chest; the key settles against the chest as the route ends.
5. **The library** — doors open onto warm light and the “Try the door” entrance.

The opening title is `data/experience.json` copy: *The Lost Memories*, “Every memory has a story. Every story has a place.” and “Follow the map. Discover what was left behind.” The header offers sound (off by default), the field journal and **Skip the journey**. The journey is remembered in `localStorage`; returning visitors go straight to the door. `?intro` forces the full journey again.

Sound is synthesized with WebAudio: a quiet drone, filtered noise and a few bell tones. Nothing is downloaded and nothing plays until the visitor turns it on.

### Clues and the library door

Four brass marks sit at the compass, the bridge, the mansion door and the iron key. Inspecting one opens its inscription and adds it to the field journal. The inscriptions are atmospheric and **do not contain the library password**; they point to the keeper, not to a word. They are fully editable in `data/experience.json`:

```json
{
  "id": "compass",
  "scene": "map",
  "title": "The brass compass",
  "inscription": "It does not point north. It points to what was left behind.",
  "fragment": "I"
}
```

The real library word is checked only by `POST /api/unlock/library`; no password or hash is present in the HTML, CSS or JavaScript. The gate also explains that an authorised visitor who already knows the word does not need the journal.

### The library and five volumes

After the library word, the door opens onto a warm walnut library with five books:

| Volume | Leather |
| --- | --- |
| I — Krishna | royal blue + antique gold |
| II — Hayson | dark emerald + antique gold |
| III — Nilesh | deep burgundy + antique gold |
| IV — Lizbeth | dusty plum + antique gold |
| V — Library Book | dark walnut + antique gold |

Books lift from the shelf, cast a deeper shadow and carry a soft reflected sheen that follows the cursor. Gold lettering brightens on hover, and a small “Open this memory” label appears. Clicking slides the chosen book forward, then asks for that volume's password. On touch screens the same states are driven by tap and focus instead of hover.

Each personal volume opens with **The Story of …** before the album. It shows only verified facts from the archive: the folder the volume came from, the number of photographs and films, and date ranges that survived on the files. There is no invented biography, relationship or event. A volume with no photographs gets an honest empty-volume page rather than placeholder imagery.

The fifth **Library Book** (slug `shared`) contains the group folder: group photographs, shared films, unassigned frames and multi-person memories. It is open once the library session is unlocked, because it is the group's own record rather than a personal secret.

### The album

- Two-page parchment spreads on desktop, a single page at ≤900 px, with swipe support
- Varied layouts: solo, pair, wide, full-spread and vintage film frames
- Mounted plates with photo corners, soft page shadows, handwritten dates only when the file supplied a plausible date
- Page numbers and “Pages X – Y of N” progress
- A 3D page-turn on desktop; a lighter single-page turn on mobile
- Click any photograph for a large viewer with previous/next, or any film poster to play it lazily
- Close returns to the library with a cover-closing animation

### Upload Treasure

At the foot of the library, **Upload new treasure** opens a private archive desk. It accepts multiple photographs and films (JPEG, PNG, WebP, HEIC, MP4, MOV; up to 200 MB each), one destination volume, an optional date and a caption.

The flow is deliberately two-step:

1. `POST /api/uploads` streams one raw file, validates its MIME type and signature, measures it, makes display/thumb derivatives (or a video poster), computes SHA-256 and holds the record in `uploads/pending/` and `data/uploads.json`. Nothing enters a book yet.
2. The pending card previews the private file. **Publish this memory** calls `POST /api/uploads/:id/publish`; **Discard** calls `DELETE /api/uploads/:id`. Only publish writes a derivative into `media/books/<volume>/`, adds the item to `data/archive.json` and reloads the catalogue in place.

Upload sessions require the library grant and same-origin mutation checks. Preview and derivative routes require the same session. Duplicate bytes are refused with `409`; the hash index includes published archive files and pending uploads. Files that fail type, size, decode or duplicate checks are rejected before they reach a book.

**AI captions:** no vision model is available in this zero-budget build, so the desk does not call one and the caption is written by the person uploading. No identity, date, relationship, place or feeling is generated. If an `OPENAI_API_KEY` is later supplied, the existing offline `npm run describe` CLI can generate editable descriptions; uploading itself stays manual and server-authorised.

### Visual descriptions

Descriptions are short, factual notes shown beneath plates and in the viewer. They are optional; a photograph with no note simply has no note.

**Generate them** (requires an OpenAI-compatible vision key; never commit the key):

```bash
printf 'OPENAI_API_KEY=...\n' > .env
npm run describe -- --dry                 # list what would be described
npm run describe -- --only=hayson --limit=10
npm run describe -- --videos              # include one poster frame per film
```

The CLI sends the actual thumbnail pixels, not filenames. Its prompt forbids invented identities, relationships, locations, dates and feelings; it asks for two sentences and a maximum of 45 words. Results are cached in `data/descriptions.json`, keyed `slug/key`, and re-runs skip completed items unless `--force` is passed. Nothing is analysed when the site opens.

**Edit by hand** in either place:

- In the album, click a photograph → **Add a description** / **Edit description** → Save or Remove. This writes through `PUT /api/books/:slug/items/:key/description`, which requires the library session and that volume's grant.
- Or edit `data/descriptions.json` directly. The server reads it on demand, so a restart is not required.

To remove every generated note, replace the `descriptions` object with `{}`. To remove one, delete its `"slug/key"` entry.

### Performance

The journey still uses native scroll, but the heavy work is now CSS state rather than per-frame JavaScript:

- the route draws with a CSS `stroke-dashoffset` transition and `pathLength="1"`
- map camera, chapter fades, chest lid, key and doors use CSS transitions driven by five phase states
- JavaScript only chooses the phase, writes the 1%-step progress bar and updates the chapter label
- no per-frame `filter: blur()` on fog, no continuous map/key animations and no style writes for elements that are not changing
- paint/layout containment is applied to the fixed scenes, and the scroll height is cached

Measured in headless Chrome with real wheel input at 1280×900: **76.2 FPS on the GPU path**, **49.1 FPS with software rendering**; p95 frame times were 12.7 ms and 50 ms respectively. The static end state held about 60 FPS. Synthetic `scrollTop` measurements were discarded because they bypass wheel-driven compositing and reported misleadingly low numbers. The scroll interaction is never hijacked.

### Accessibility and motion

- Native scroll; the journey can always be skipped
- Real buttons, visible labels, arrow-key page turns, Escape to close overlays, focus return after closing
- `prefers-reduced-motion` collapses animation/transition durations and disables the journey's ambient loops
- Sound is off by default and never autoplays

---

## 4. Media mapping

The source folder contains five subfolders. Assignment is **strictly by folder** — nothing is guessed by face or filename.

| Volume | Drive folder ID | Ingested |
| --- | --- | --- |
| Krishna | `1KNQ09hy8W5YvLAa0kH2-3LJbYiHnlpM0` | 82 photographs · 4 films |
| Hayson | `1A5_4aSaJokKz6Pl32UocaodZNLhw6eGG` | 36 photographs · 2 films |
| Nilesh | `163_Fn3HG0GpoqW7Fhf2m8nAcjxPtMUxa` | 151 photographs · 11 films |
| Lizbeth | `1ZR5m1I3c9kunyAN1wH83byCy_TslDMzS` | 63 photographs · 10 films |
| Library Book | `14tu0ed4KcC-ege5wiva4FjU_67g-eq-K` (“Anythings”) | 41 photographs · 35 films |

**Verified total: 435 files — 373 photographs and 62 films.** `data/source-listing.json` records every original filename, Drive file ID and destination volume — the audit trail for the mapping. Deduplication is by Drive file ID only: **eight duplicate filenames are distinct files** and are deliberately kept. There are zero duplicate IDs, zero unassigned files, zero missing files, zero extras and zero undecodable local files.

Re-run the audit at any time:

```bash
npm run verify:media
```

It re-lists the public Drive folders, matches by file ID, decodes every local image/video derivative, and rewrites `data/media-verification.md` and `.json`.

### Adding photographs or films

The ingest is incremental and reads the Drive folders live:

```bash
npm run media                 # all volumes
npm run media -- --only=hayson --refresh   # rebuild one volume from scratch
```

It requires ImageMagick 7 (`magick` on PATH). Video compression uses the optional `ffmpeg-static` dev dependency; without it the original MP4 is copied and still streamed with byte ranges. Existing derivatives are skipped.

What it produces per item:

- `display` JPEG — longest edge 1600 px, quality 82, EXIF stripped, orientation applied
- `thumb` JPEG — longest edge 560 px (film posters 1400 px)
- films — H.264/AAC at max 1280 px wide, `faststart`, only if smaller than the original

Originals in Drive are never modified or deleted.

**Captions** — `data/captions.json`:

```json
{
  "captions": {
    "hayson/20260911_172110.jpg": "The first evening in Goa.",
    "shared/VID-20261004-WA0000.mp4": "Everyone, briefly still."
  }
}
```

Re-run `npm run media` after editing; captions are copied into the manifest. Dates are read from filenames first, then EXIF, and implausible camera-clock dates (e.g. 2000-01-01) are treated as undated rather than shown as fact.

---

## 5. Authentication and privacy

This is **real server-side access control**, not a JavaScript password screen.

- **Passwords** — scrypt (N=16384, r=8, p=1), 16-byte salt, stored only as hashes in a gitignored file.
- **Sessions** — a 32-byte opaque token in an `HttpOnly`, `SameSite=Strict` cookie (adds `Secure` over HTTPS). Session records live on the server; the cookie carries no identity or grant.
- **Grants** — the library session is distinct from a book grant. Opening Nilesh's volume issues a grant for `nilesh` only.
- **Media** — `/media/:book/:key` checks the session and the book grant before touching the filesystem. Paths come from the manifest by key, never from the URL. `Cache-Control: private, no-store`.
- **Descriptions** — write access requires both the library session and the volume grant; reads are merged only into an already-authorised book response.
- **CSRF** — `SameSite=Strict` plus an Origin/Host check on every state-changing request.
- **Throttling** — 8 attempts per IP per 10 minutes for the library and for each volume.
- **Headers** — CSP (self only), `X-Frame-Options: DENY`, `noindex`, `nosniff`, `no-referrer`.
- **Logout** — “Lock the archive” destroys the server session and cookie.

### Honest limitations

- The five passwords are shared secrets. There are no per-person accounts, email recovery or audit of who viewed what.
- Sessions and rate limits are in-process memory: a restart signs everyone out, and multiple server instances do not share them. Run one instance, or add a shared session store.
- Anyone who knows a volume password can view that volume. Anyone with server or disk access can read the media.
- **GitHub-only hosting cannot run this archive securely.** GitHub Pages serves every file publicly, so a password check there would be decorative and every photograph would be downloadable. GitHub can host the source repository; a Node process with a persistent disk must serve the archive. No paid host, database, storage bucket or AI service has been added.
- **AI caption generation is not available offline.** There is no `OPENAI_API_KEY`, local vision model or Ollama in this build, so the upload desk asks the owner to write the caption and clearly says so. The UI never fabricates a caption and never pretends a model ran.
- The upload queue and archive manifest are files on the Node host's disk. A multi-instance or serverless deployment would need shared storage and a database; that is intentionally not faked here.

---

## 6. Verification performed

- Library password accepted; wrong password rejected with a server 401.
- Session cookie alone does not grant a personal volume: `/api/books/hayson` → **403** until that volume is unlocked.
- `/media/<book>/<key>` → **401** without a session, **403** for a different volume's grant.
- **435/435 Drive files verified** by ID: 373 photographs, 62 films, 0 missing, 0 extras, 0 corrupted, 0 unassigned and 0 duplicate IDs. Eight duplicate filenames are distinct files and are kept.
- `/`, `/library` and all five `/books/:slug` routes return 200; direct reload, browser Back and deep links were checked.
- Hayson and the Library Book opened end to end in the current build. All five story routes load and all five volume counts/media are covered by the 435-file audit; Nilesh, Krishna and Lizbeth remain password-gated and were not reopened in the final pass because their passwords were not available in this session.
- Shared Library Book opens without a personal password; films render as lazy vintage frames with poster thumbnails.
- A Hayson film loaded to `readyState 4`, played to 1.26 s, and reported a 4.91 s duration; video responses support byte ranges (`206 Partial Content`).
- Page turning, page numbers and progress checked on desktop and at 390 px width (single-page layout, no horizontal overflow).
- Photograph viewer opens with its counter and previous/next controls.
- Upload desk exercised end to end with a generated test image: preview, pending card, destination/caption/date fields, duplicate/size/type validation and explicit discard. The test record and derivatives were removed afterwards.
- Journey performance measured with real wheel input: 76.2 FPS GPU / 49.1 FPS software; no horizontal overflow at 390 px; no console exceptions in the album/upload runs.
- Description save and remove verified through the viewer; `data/descriptions.json` was written and cleared atomically. Generation itself is blocked until `OPENAI_API_KEY` is supplied.
- “Lock the archive” clears the session: `/api/session` reports `unlocked: false`, `/api/books/hayson` → **401** and protected media → **401**.
- Clue inspection increments the field journal and reveals only inspected inscriptions; unfound marks stay unread.
- No console errors; fonts and media return 200 only through the authenticated routes.

Screenshots from the final desktop/mobile pass are in `screenshots/`; the accompanying DOM/API checks were run directly against the same server.

---

## 7. Updating the archive

| Change | Action |
| --- | --- |
| Add one file through the site | Library → Upload new treasure → choose files/volume → Publish this memory |
| Add photos/films to a Drive folder | `npm run media` |
| Audit Drive against local media | `npm run verify:media` |
| Generate photo descriptions | set `OPENAI_API_KEY`, then `npm run describe` |
| Edit or remove one description | viewer → Edit description, or edit `data/descriptions.json` |
| Rename a volume or change leather colour | edit `data/books.json`, restart the server |
| Change journey/gate copy or clue inscriptions | edit `data/experience.json` |
| Add captions | edit `data/captions.json`, `npm run media` |
| Change a password | `npm run passwords` (no restart needed) |
| Force the intro for returning visitors | open `/?intro` |
| Reset the intro memory | `localStorage.removeItem('fourOfUs.journeySeen')` |
| Update CSS/JS | no build step; bump the `?v=` query in `public/index.html` and `public/js/app.js` |

The final audit, performance numbers, upload checks and remaining limitations are in **[REVISION-REPORT.md](REVISION-REPORT.md)**. Deployment, HTTPS, backups and static-host caveats: **[DEPLOYMENT.md](DEPLOYMENT.md)**.
