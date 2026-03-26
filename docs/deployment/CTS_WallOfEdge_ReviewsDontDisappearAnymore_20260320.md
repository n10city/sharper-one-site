# CTS™ — Reviews Don't Disappear Anymore
**Date:** 2026-03-20 · **Thread:** Wall of Edge™ · Server-Side Persistence · Full Deploy

---

## STEP 1 — CLOSE

### What Got Built
- **`submit.php`** — receives POST from `/edge/`, validates, sanitizes, appends to `data/entries.json` flat store. Strips base64 photos (future build). Returns `{success, id, message}`.
- **`entries.php`** — serves `entries.json` as JSON. Public mode strips `notes` and filters `isPublic: false`. Internal mode gated by `X-WOE-Mode: internal` + `X-WOE-Secret` header pair.
- **`wall-of-edge.html`** — `load()` now async, server-first fetch from `entries.php`, localStorage as offline/fallback cache. `toggleInternalMode()` reloads from server with correct headers. `saveEntry()` optimistic local update + async POST to `submit.php`.
- **`edge.html`** — `submitEntry()` now async, POSTs to `/wall/submit.php`. Falls back to localStorage write if server unreachable — customer never sees a broken experience.
- **`fonts.css`** — created from scratch, all 8 woff2 variants declared (`Bebas Neue`, `Barlow Condensed` 400/600/700, `Barlow` 400/italic/600/700). Deployed to `/fonts/`.
- **`barlow-v13-latin-italic.woff2`** — corrupt file replaced with valid font pulled server-side via curl from `fonts.gstatic.com`.
- **Nav orb** — Wall nav orb swapped from CSS `<div>1</div>` to real `<img>` asset (`/icons/Sharper-ONE_Logo-Symbol_app-icon_trans.png`), matching main site pattern.

### Repo Housekeeping Sealed
- Stray root `index.html` (duplicate of `src/index.html`) — deleted
- `server/wall/edge.html` + `server/wall/wall-of-edge.html` (session artifacts) — deleted
- `CTS_WallOfEdge_OrbFoundItsHome_20260320.md` — moved to `docs/architecture/`
- `server/wall/` directory created — `submit.php`, `entries.php`, `README.md` tracked in repo
- `public/fonts/fonts.css` + `barlow-v13-latin-italic.woff2` committed

### Decisions Locked
- PHP files live in `server/wall/` in repo — deploy via SCP, never through npm build
- `data/entries.json` lives in `public_html/wall/data/` — locked with `Deny from all` in `.htaccess`
- `WOE_SECRET = 'sharper1edge'` — must match in both `entries.php` and `wall/index.html`
- Base64 photo storage intentionally stripped at `submit.php` — flat JSON protection. Future: `/wall/upload.php` → `/data/photos/[id]/` → URL stored in entry
- Server-side curl (not WSL curl) is the reliable method for pulling woff2 files from `fonts.gstatic.com`
- `unset GIT_DIR && unset GIT_WORK_TREE` no longer needed — Logseq Git plugin confirmed disabled, `~/.bashrc` confirmed clean
- Never `scp` from server back to repo to "sync" — server may have old version; always verify with `grep` first

### Commits This Thread
| Hash | Message |
|------|---------|
| `2ce00bd` | fix: logo orb — transparent fill, orange border trim, pulse on border |
| `512f2b7` | feat: Wall of Edge™ — server-side persistence (PHP files + README) |
| `c9779ef` | fix: wall — deploy server-first fetch version + CTS archived |
| `[font commit]` | fix: replace corrupt barlow italic woff2 with valid font file |
| `[orb commit]` | fix: wall nav orb — swap CSS div for real ONE™ logo asset |

### The Proof
- Submission from Samsung Galaxy → `entries.json` written on server → Wall on desktop pulls live from `entries.php` → entry appears. Cross-device. Cross-browser. Persistent.
- Reviews don't disappear anymore. Ever.

---

## STEP 2 — TRANSITION

**New thread mission:** Migrate seed entries to server-side JSON + real end-to-end customer submission test (phone → wall, not curl → wall)

**Immediate objectives in sequence:**
1. Delete test entry (`woe_1774026582_81dc7504`) from `entries.json` on server
2. Migrate the 3 seed entries (Maria T., DeShawn B., Jericho L.) to `entries.json` — these should survive as permanent Wall fixtures
3. Submit a real customer-style entry from phone via `sharper.one/edge/`
4. Confirm it appears on Wall on desktop without clearing localStorage
5. Test Internal View toggle — confirm ops notes visible, non-public entries visible

**Carried-forward context:**
- Server: Vultr · IP: `155.138.200.125` · Enhance: `engine.i2i.host` · Apache
- SSH alias: `sharper-one` · web root: `/var/www/e1508a19-43fd-42c4-97a1-958e8b5e6763/public_html/`
- `/start` — live and protected, never touched during deploys
- `WOE_SECRET = 'sharper1edge'` — change before any public internal-mode use
- Photo persistence — still localStorage/null. Future thread: `/wall/upload.php`
- `data/entries.json` path: `/var/www/e1508a19-43fd-42c4-97a1-958e8b5e6763/public_html/wall/data/entries.json`

**Standing items for future threads:**
- Wall of Edge™ admin protection — `.htaccess` Basic Auth on `/wall/` internal mode (ops.sharper.one pattern)
- Change `WOE_SECRET` from default before trade outreach begins
- Photo upload endpoint — `/wall/upload.php` → `/data/photos/` → URL in entry
- Intake Cards — field-ready, printable, TiO™ standard
- Client Communications — transaction confirmations, onboarding series, nurture flow
- Payment process testing — Square walk-up + Stripe SEC™ membership verification
- VSCode `.code-workspace` file — create and commit to repo
- `CLAUDE.md` — font policy section needs to be added
- `go.sharper.one` — Shlink deployment
- Brevo integration — main site sticky bar + membership modal
- Google Business Profile — review collection configuration
- Customer Journey Card — station display + digital distribution

---

*CTS™ · The SomeBody™ Company · BiMKA™ · EST™*
*See yuh OTOS™* 🔱
