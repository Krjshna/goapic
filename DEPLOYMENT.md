# Deploying The Lost Memories

This archive is a Node.js server, not a static site. The library door and the media protection only exist because the server refuses to send a photograph without a valid session and the matching volume grant. The treasure-map journey and the five-volume library are served by the same Node process.

## Why GitHub Pages cannot host this archive on its own

GitHub-only hosting is not sufficient here, and the project does not pretend otherwise. GitHub Pages serves files directly. A password prompt written in JavaScript is a decoration: the HTML, the JavaScript and every image file remain fetchable by anyone who views source, guesses a path or opens the DevTools network tab. There is no server to check a password and no way to hide the media. Uploading this front end to Pages would publish the whole archive.

The source repository can live on GitHub. The running archive needs a Node host with a persistent disk for `media/`, `config/`, `data/descriptions.json` and the pending upload queue. Until such a host is chosen, there is **no public deployment** of this archive; that is intentional, not a missing deploy step. No paid service, external database, object bucket or AI API has been added.

### What is impossible with GitHub alone and zero budget

The following cannot be delivered securely or honestly by GitHub Pages alone:

- password-checked pages and private photographs — the files would be public static assets
- private, reviewed uploads — there is no server-side validation, queue, hash check or publish step in static hosting
- server-side AI captioning — there is no secret key or model runtime, and a browser key would be public
- durable captions, archive edits or upload metadata — there is no writable database or disk

The repository therefore keeps the Node server in front of the media. Do **not** upload `media/`, `config/`, `data/uploads.json` or `uploads/` to a public Pages project or a public Git history.

---

## What the host must provide

- **Node.js 20.11+** and a start command (`npm start`)
- **A persistent volume** containing:
  - `media/` — the derivatives (several hundred MB)
  - `config/archive.config.json` — the password hashes
  - `data/archive.json`, `data/descriptions.json` and the writable `uploads/` + `data/uploads.json` queue
- **HTTPS** — session cookies become `Secure` automatically when `X-Forwarded-Proto: https`, and `trust proxy` is enabled
- **One instance** — sessions and the attempt limiter are in memory. Horizontal scaling would need a shared store (Redis) and sticky sessions; not included.

ImageMagick and ffmpeg are **not** runtime requirements. They are only needed on the machine that runs `npm run media` or prepares upload derivatives. Run the ingest locally, then upload the resulting `media/` directory.

---

## Deploy

### 1. Prepare the artifact locally

```bash
cd four-of-us
npm ci --omit=dev          # Express only; ffmpeg-static is a dev dependency
npm run media              # if the Drive contents changed (needs ImageMagick)
npm run passwords          # set the real five passwords
```

Commit or upload:

- everything except `node_modules/`, `data/.media-cache.json`
- **including** `media/` and `data/archive.json`
- **not** `config/archive.config.json` into a public repository. Copy it to the server privately, or run `npm run passwords` on the server after the first deploy.
- **not** `uploads/`, `data/uploads.json` or `data/backups/` — these are private and gitignored

### 2. Environment

| Variable | Production value |
| --- | --- |
| `PORT` | the port your platform assigns (often `10000` or `8080`) |
| `HOST` | `0.0.0.0` |
| `NODE_ENV` | `production` |

`OPENAI_API_KEY` is **not** a runtime variable. It is only read by `npm run describe` on the machine that generates `data/descriptions.json`. Keep it in `.env` (gitignored), never in `public/` or a client bundle.

The upload desk does not call an AI model. In a zero-budget deployment without a vision key or local model, the caption is written by the owner and no caption is invented. The UI states this explicitly; do not add a browser-side API key to change that.

### 3. Start

```
npm start
```

Health check: `GET /api/session` returns JSON with `configured: true` and HTTP 200. It is public by design and contains no media.

### 4. Platform notes

- **Render / Railway / Fly.io** — a normal Node web service plus one persistent volume mounted at the project root (or at `media` and `config`). Keep the instance count at 1.
- **VPS** — run behind Nginx or Caddy with HTTPS, proxy to `127.0.0.1:4310`, and set `HOST=127.0.0.1` again so Node is not exposed directly. Use a process manager (systemd, pm2) with a restart policy.
- **Docker** — start `node server/index.js` as PID 1, mount volumes for `media` and `config`, expose the port. No build step is needed.

### 5. After the first deploy

1. Open the site over HTTPS.
2. Enter the library password, then each volume password.
3. Check that a direct media URL in a signed-out browser returns `401`, and that another volume's key returns `403`.
4. Open Upload new treasure, place a small test file in the pending tray, confirm the private preview, then discard it. Check that it never appears in the public archive.
5. Put the password file, `media/`, `data/archive.json` and the private upload queue into your normal backup routine. Losing `config/archive.config.json` means re-running `npm run passwords`; losing `media/` means re-running `npm run media`.

---

## Rotating passwords

```bash
npm run passwords
```

The new hashes are read on the next unlock attempt; no restart is needed. Existing sessions keep their current grants until they expire, so use **Lock the archive** and/or restart the process after a credential change if you want everyone signed out immediately.

Sessions expire after 3 hours idle or 12 hours absolute. Restarting the server clears all sessions.

---

## Updating the archive

```bash
# add new photographs/films to the Drive folders first, then:
npm run media
```

Upload the changed files in `media/books/<volume>/` and the regenerated `data/archive.json`. Nothing else changes. The server does not cache the manifest in memory, so a restart is only needed for `data/books.json` edits (titles, colours, epigraphs).

If you edit CSS or JavaScript, redeploy the file; assets are served `no-store` so visitors receive the new version.

---

## Security checklist

- [ ] HTTPS only; HTTP redirects to HTTPS at the proxy
- [ ] `config/archive.config.json` is outside version control and file permissions are owner-only
- [ ] `uploads/`, `data/uploads.json` and `data/backups/` are outside version control
- [ ] `media/` is not in `public/` and is never served by a second static handler
- [ ] No directory listing is enabled by the reverse proxy
- [ ] Reverse proxy passes `X-Forwarded-For` and `X-Forwarded-Proto`
- [ ] Backups of both `media/` and the password config exist
- [ ] The five passwords are unique, long, and not reused from anywhere else

## If you need more than shared passwords

The current model is a private family archive with five shared secrets. For stronger guarantees — individual accounts, revocation, view history — put an identity provider in front (OIDC), store media in a private bucket, and serve short-lived signed URLs after authorisation. The album front end can stay; only the server's session and media routes change.
