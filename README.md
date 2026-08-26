# donna-browser

Playwright Chromium sidecar for [Donna](https://github.com/kishansagathiya/donna).

The Go server (`donna-server-go`) calls this service when `DONNA_BROWSER_URL` is set. Chat uses one-shot `browse_page`. Cloud agents use a **session per agent run** (`/session/*`) so they can click and type on the same page. Without the sidecar, Donna still has `fetch_url` (plain HTTP + HTML→text).

## Run locally

```bash
npm install
npm run install-browsers
npm start
```

Listens on `http://127.0.0.1:9229` by default (`DONNA_BROWSER_HOST` / `DONNA_BROWSER_PORT`).

Point Donna’s server at it:

```bash
export DONNA_BROWSER_URL=http://127.0.0.1:9229
```

## API

| Method | Path | Body / response |
|--------|------|-----------------|
| `GET` | `/health` | `{ ok, service, active, sessions }` |
| `POST` | `/browse` | `{ url, wait_ms?, max_chars? }` → `{ url, title, text, status }` (new context, then close) |
| `POST` | `/session/navigate` | `{ session_id, url, wait_ms? }` → snapshot (`url`, `title`, `text`, `elements[]` with `ref`) |
| `POST` | `/session/snapshot` | `{ session_id }` → tagged interactive elements (`data-donna-ref`) |
| `POST` | `/session/click` | `{ session_id, ref }` → snapshot after click |
| `POST` | `/session/type` | `{ session_id, ref, text, submit? }` → snapshot after fill |
| `POST` | `/session/extract` | `{ session_id, max_chars? }` → `{ url, title, text }` |
| `POST` | `/session/close` | `{ session_id }` → `{ ok }` |

Sessions are keyed by `session_id` (Donna uses the `agent_run` id). Idle sessions close after 15 minutes. Max 8 concurrent sessions. Public-URL checks happen in the Go client, not here.

Interactive `elements` look like `{ ref, tag, type, name, role }` (e.g. `ref: "e3"`). Agents must `snapshot` again if a click navigates.

## Railway / production

Deploy as its own service, then set on `donna-server-go`:

```bash
DONNA_BROWSER_URL=https://your-donna-browser.up.railway.app
```

Prefer private networking between Railway services when possible. Bind with `DONNA_BROWSER_HOST=0.0.0.0` and Railway’s `PORT`.

## Related

- Monorepo: https://github.com/kishansagathiya/donna
- Server: https://github.com/kishansagathiya/donna-server-go
- Plan: `docs/improvement-plans/02-chat-browser-tool.md` in the monorepo
