# The Four of Us — Revision Report

**Date:** 9 November 2026  
**Scope:** media integrity, animation performance, addressable pages, uploads, captions, zero-budget/GitHub-only constraints, verification.

## 1. Media — verified

`npm run verify:media` re-listed all five Google Drive folders and compared them to the local derivatives by **Drive file ID**:

| Volume | Photographs | Films | Total |
| --- | ---: | ---: | ---: |
| Krishna | 82 | 4 | 86 |
| Hayson | 36 | 2 | 38 |
| Nilesh | 151 | 11 | 162 |
| Lizbeth | 63 | 10 | 73 |
| Library Book (shared) | 41 | 35 | 76 |
| **Total** | **373** | **62** | **435** |

Result: **435 matched · 0 missing · 0 extra · 0 corrupted · 0 unassigned · 0 duplicate IDs.**

Eight duplicate **filenames** exist; they have distinct Drive IDs and are therefore distinct files, not accidental copies. Deduplication is by ID only. The durable reports are `data/media-verification.json` and `data/media-verification.md`. Pre-repair data and the source listing are in `data/backups/`.

The earlier shared-volume test film was removed; the archive is back to its verified 435 files.

## 2. Animation and performance

The journey was moved from per-frame JavaScript styling to CSS phase transitions:

- route uses CSS stroke drawing with `pathLength="1"`
- map camera, chapter fades, chest lid/key and library doors are CSS transitions
- JavaScript selects the phase and writes only a 1%-step progress bar
- fog blur filters and continuous ambient animations were removed
- fixed scenes use paint/layout containment; scroll height is cached
- native scrolling remains native; no scroll hijacking

Measured with real wheel input in headless Chrome at 1280×900:

| Path | Result |
| --- | --- |
| GPU | **76.2 FPS**, p95 **12.7 ms** |
| Software rendering | **49.1 FPS**, p95 **50 ms** |
| Static end state | ~60 FPS |

Synthetic `scrollTop` measurements were discarded because they bypass wheel-driven compositing.

## 3. Pages and navigation

Routes are real and addressable:

- `/` — journey / gate / library, depending on state
- `/library` — bookshelf and archive desk
- `/books/:slug` — one volume's story page

Tested: HTTP 200 for `/`, `/library`, and all five `/books/:slug`; direct reload; History API Back/Forward; signed-out deep links stop at the gate; sealed volumes stop at their password card; unknown slugs return to the library with a notice. Library and album were checked at 1280 px and 390 px with no horizontal overflow.

## 4. Upload Treasure — implemented and tested

The library desk accepts multiple photographs/films, a destination volume, an optional date and a caption.

- raw upload with 200 MB per-file limit
- MIME/signature validation and image/video probe
- private pending storage in `uploads/pending/` + `data/uploads.json`
- SHA-256 duplicate refusal (published archive + pending queue)
- private preview, derivative and poster generation
- explicit **Publish this memory** and **Discard**
- publish writes into `media/books/<volume>/`, updates `data/archive.json` and reloads the catalogue without a restart
- library-session and same-origin checks on every mutation

End-to-end browser test used a generated, non-personal test image. The preview, pending card, destination/caption/date controls and discard all worked; the test record and files were removed afterwards. Queue and pending directory are empty.

## 5. AI captions — honest limitation

No `OPENAI_API_KEY`, Ollama or local vision model is available in this zero-budget environment. The upload desk therefore does **not** call an AI service and says so in the form: the caption is written by the uploader. No identity, date, relationship, place or feeling is generated.

The existing `npm run describe` CLI can generate editable, cached descriptions if an OpenAI-compatible key is supplied later. It is offline-only and not part of the public runtime.

## 6. GitHub-only / zero budget

No GitHub deployment was made. GitHub Pages cannot securely provide:

- password-checked private photographs (all files would be public)
- private pending uploads and server-side validation
- server-side AI captioning
- durable archive/caption/upload writes

The Node server is retained precisely so those features are not faked. The source can be committed to GitHub, but the running archive needs a Node process with a persistent disk. No paid host, external database, object bucket or AI service was added.

## 7. Checks run in the final pass

- media audit: **435/435**
- route checks: `/`, `/library`, all five books
- auth: library login 200; Hayson grant 200; sealed Krishna 403; unauth APIs/media 401; foreign origin 403
- media: private photo 200, private video `206` byte range, Hayson film loaded to `readyState 4` and played to 1.26 s
- upload: real browser upload → pending card → discard; no residue
- browser: Hayson story and album, film playback, mobile album and upload desk, no console exceptions
- responsive: `scrollWidth === innerWidth` at 390 px for library/upload and album

**Not completed:** a fresh visual pass of Nilesh, Krishna and Lizbeth albums (their volume passwords were not available in this session). Their routes, counts, files and media integrity are verified; Hayson and the Library Book provided the end-to-end album/playback coverage.

## 8. Run and verify

```bash
cd four-of-us
npm install
npm start                 # http://127.0.0.1:4310
npm run verify:media      # re-audit Drive against local derivatives
```

Deployment constraints and the secure-host checklist are in `DEPLOYMENT.md`.
