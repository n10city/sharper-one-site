# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Mission

Deploy the Sharper ONE™ landing page at https://sharper.one/start. Current phase is **Launch Deployment** — not design or strategy. Do not suggest scope expansion or redesign until the page is publicly live.

Read `_AI/ACTIVE_STATE.md` at the start of any session for current operational state.

## Commands

```bash
npm install       # Install dependencies (first time or after package.json changes)
npm run dev       # Start Astro dev server → http://localhost:4321/start/
npm run build     # Production build → /tmp/sharper-one-build/
npm run preview   # Preview production build locally
```

No test suite is configured.

## WSL + NTFS Build Setup

This project lives on a Windows drive (`/mnt/d/`). Vite's `copyFileSync` fails on NTFS (EPERM — Node.js uses FICLONE flags unsupported by NTFS). Three workarounds are baked in:

1. **`.npmrc`** — `bin-links=false` prevents npm from `chmod`-ing CLI binaries. Scripts invoke Astro via `node node_modules/astro/astro.js` directly.
2. **`prebuild` script** — copies `public/` to `/tmp/sharper-one-public/` and symlinks `.astro/` → `/tmp/sharper-one-astro/` so all Vite write ops land on ext4.
3. **`fixWslModuleResolution` Vite plugin** (`astro.config.mjs`) — after the SSR bundle is written to `/tmp/sharper-one-astro/`, creates a `node_modules` symlink there so ESM resolution works when Astro runs the SSR chunks to generate static HTML.

**Build output lives at `/tmp/sharper-one-build/`** (wiped on WSL/system restart). Run `npm run build` again if it's gone.

**Deploy command** (after build):
```bash
scp -r /tmp/sharper-one-build/* sharper_1@155.138.200.125:/var/www/e1508a19-43fd-42c4-97a1-958e8b5e6763/public_html/start/
```

## Architecture

This is a static Astro + Tailwind site (SSG, no React, no framework JS).

**Stack:** Astro 5, Tailwind CSS 3, `@astrojs/tailwind` integration, Inter font (Google Fonts)

**Key files:**
- `src/pages/index.astro` — The entire landing page. Single source of truth for all content and UI.
- `src/layouts/Layout.astro` — Base HTML shell: meta, OG tags, Inter font, body wrapper.
- `astro.config.mjs` — `output: 'static'`, `base: '/start'` (required for correct asset paths on the server), Tailwind integration.
- `tailwind.config.mjs` — Content paths, Inter font family extension.

**UI patterns:**
- No component library. All UI is native HTML + Tailwind utility classes.
- Icons are inline SVG, no icon library.
- Booking modal uses native `<dialog>` + vanilla JS (`showModal()` / `close()`).
- `?live` URL param shows a live-day banner via an inline `<script>` at page load.
- Stripe Payment Link is a plain `<a href="{{STRIPE_LINK}}">` — swap before deploy.
- Phone placeholder is `{{PHONE}}` in `tel:` hrefs — swap before deploy.

## Font Policy — TiO™ Standard
- All fonts self-hosted under /public/fonts/ — no external CDN links in deployed files
- Google Fonts links are dev-only convenience — strip before any deploy
- font-display: swap required on every @font-face declaration
- Shared font path: /fonts/ resolves from sharper.one root — all subpages share one copy
- New fonts: download from gwfh.mranftl.com, woff2 format only

## Deployment Target

- **Server:** moca-prod / i2i HOST (`155.138.200.125`)
- **Control plane:** Enhance (LiteSpeed, not Apache)
- **Website ID:** `e1508a19-43fd-42c4-97a1-958e8b5e6763`
- **Content root:** `/var/www/e1508a19-43fd-42c4-97a1-958e8b5e6763/public_html`
- **Launch path:** `/start` → public URL: `https://sharper.one/start`
- **Do NOT deploy to** miller-prod (`66.42.93.230`) — legacy server only

**Build pipeline:** `npm run build` → `dist/` → upload `dist/` contents to `/public_html/start/`

The `base: '/start'` in `astro.config.mjs` ensures that bundled asset paths (e.g. `/_astro/index.hash.css`) are prefixed with `/start/`, so they resolve correctly when the files live inside the `/start/` subdirectory on the server.

**Before deploy — swap these placeholders in `src/pages/index.astro`:**
- `{{STRIPE_LINK}}` → Stripe Payment Link URL
- `{{PHONE}}` → business phone number (used in `tel:` hrefs)

## _AI/ Reference Files

| File | Purpose |
|------|---------|
| `ACTIVE_STATE.md` | Operational front door — read first each session |
| `MISSION_LOCK.md` | Locked mission constraints |
| `PROJECT_COMPASS.md` | Project type and goals |
| `SERVER_MAP_MOCA-PROD.md` | Server infrastructure details |
| `CTS_PROTOCOL.md` | Session closure format (Capture → Transfer → Start) |
| `AI_SESSION_START.md` | Session startup checklist |

## Operating Principles

- Speed and clarity over elegance during launch
- Treat existing project decisions as locked unless explicitly questioned
- Prefer execution steps over explanations
- Do not re-scope the mission if context appears missing — ask a precise question instead

---

## SOIL™ System — Sharper ONE™ Intake LiFE™

> **What this is:** A two-device intake pipeline for pop-up sharpening sessions. Operator device creates sessions; customer device completes them via QR link. Data feeds Wall of Edge™ and downstream communication systems.

---

### Server Paths (Vultr / Enhance / Apache)

| Resource | Path |
|----------|------|
| Operator page | `/var/www/e1508a19-43fd-42c4-97a1-958e8b5e6763/public_html/intake/index.html` |
| Customer page | `/var/www/e1508a19-43fd-42c4-97a1-958e8b5e6763/public_html/intake-c/index.html` |
| PHP — create | `/var/www/e1508a19-43fd-42c4-97a1-958e8b5e6763/public_html/intake/session_create.php` |
| PHP — read | `/var/www/e1508a19-43fd-42c4-97a1-958e8b5e6763/public_html/intake/session_read.php` |
| PHP — update | `/var/www/e1508a19-43fd-42c4-97a1-958e8b5e6763/public_html/intake/session_update.php` |
| Sessions dir | `/var/www/e1508a19-43fd-42c4-97a1-958e8b5e6763/public_html/intake/sessions/` |
| Photos dir | `/var/www/e1508a19-43fd-42c4-97a1-958e8b5e6763/public_html/intake/sessions/photos/` |
| Auth file | `/var/www/e1508a19-43fd-42c4-97a1-958e8b5e6763/public_html/intake/.htpasswd` |
| Public operator URL | `https://sharper.one/intake/` |
| Public customer URL | `https://sharper.one/intake-c/?s=[token]` |

**Deploy pattern (SCP from WSL):**
```bash
# Operator page
scp /path/to/local/intake/index.html sharper-one:/var/www/e1508a19-43fd-42c4-97a1-958e8b5e6763/public_html/intake/index.html

# Customer page
scp /path/to/local/intake-c/index.html sharper-one:/var/www/e1508a19-43fd-42c4-97a1-958e8b5e6763/public_html/intake-c/index.html

# PHP files
scp /path/to/local/intake/session_*.php sharper-one:/var/www/e1508a19-43fd-42c4-97a1-958e8b5e6763/public_html/intake/
```

SSH alias: `sharper-one` → user `sharper_1`, key `~/.ssh/sharper_one_deploy`

---

### Session JSON Schema
```json
{
  "token": "ses_[unix_timestamp]_[4-byte-hex]",
  "firstName": "STRING — operator-entered, uppercased",
  "lastName": "STRING — customer-entered",
  "bladeCount": "INTEGER",
  "cardColor": "STRING — hex e.g. #fde8d6",
  "cardColorName": "STRING — e.g. Peach",
  "date": "STRING — e.g. Mar 21, 2026",
  "location": "STRING — e.g. Front of the Farm",
  "bladeTypes": ["ARRAY of strings — e.g. Field Blade, Chef Knife"],
  "consent": "STRING — text | email | none",
  "status": "STRING — pending | completed",
  "createdAt": "UNIX timestamp",
  "completedAt": "UNIX timestamp | null",
  "photoUrl": "STRING — relative URL path, base64 stripped at server | null"
}
```

Token format: `ses_[timestamp]_[4-byte-hex]` — short code = last 6 chars of token.
Session auto-expires at 4 hours. `session_read.php` self-cleans on read after expiry.
Sessions directory locked: `Require all denied` — never publicly accessible.

---

### Basic Auth Pattern (Operator Gate)

`.htaccess` structure:
```apache
AuthType Basic
AuthName "Operator Access"
AuthUserFile /var/www/e1508a19-43fd-42c4-97a1-958e8b5e6763/public_html/intake/.htpasswd
Require valid-user

<FilesMatch "\.php$">
    Require all granted
</FilesMatch>
```

**LOCKED DECISION:** Customer page lives at `/intake-c/` — NOT inside `/intake/`. Enhance Apache 2.4 does not honor child `.htaccess` `Require all granted` override of parent Basic Auth. This is permanent. Do not attempt to nest customer page inside `/intake/`.

**htpasswd generation (server-side only — local WSL sudo unavailable):**
```bash
ssh sharper-one "htpasswd -nb operator [password]"
```

**Group ownership:** Both `/intake/` and `/intake-c/` require `group 33` (web server) for PHP session writes. Set via root SSH (`moca-prod` alias).

---

### PHP Runtime Notes

- PHP runs as `sharper_1` — NOT `www-data`. No `chown` needed for session writes.
- Photo stored as URL path in session JSON. Base64 stripped server-side — flat JSON only.
- `session_update.php` customer completion is best-effort POST. Silent fail by design.

---

### SOIL™ Operator Page — Flow Architecture

1. **Pre-fill** — First name, last initial, blade count → populates `SESSION` object
2. **Photo + Color** — Camera capture + card color chip selection → `SESSION.cardColor` + `SESSION.cardColorName` set here
3. **QR Screen** — `goStep3()` fires → calls `session_create.php` → renders QR (qrcode.js) → shows session dot + short code + expiry

---

### ACTIVE BUG — SESSION STATE RACE (Priority 1)

**Symptom:** Operator page fails to advance Step 2 → Step 3. Zero network requests on failure. Succeeds after unpredictable number of retries.

**Confirmed exit point** in `goStep3()`:
```javascript
if (!SESSION.cardColor) return;
```
`SESSION.cardColor` is `null` when `goStep3()` fires despite operator visually selecting a color.

**Root cause theory:** `newSession()` resets `SESSION.cardColor = null`. Suspected trigger: failed attempt leaves catch block mid-execution; operator taps button before JS re-initializes; SESSION partially reset. Secondary: `show()` re-render may desync visual state from SESSION object.

**Diagnostic patch — add immediately before the guard:**
```javascript
console.log('goStep3 fired. cardColor:', SESSION.cardColor, 'cardColorName:', SESSION.cardColorName);
if (!SESSION.cardColor) { console.error('BLOCKED: cardColor is null'); return; }
```

**Fix paths (pending diagnostic confirmation):**
- Re-read `SESSION.cardColor` from DOM (selected chip's `data-color`) immediately before the guard
- Debounce button to prevent re-entry during async operations
- Prevent `newSession()` from firing except on explicit "New Customer" action

**Field data:** Paul Bragg ~3 attempts, Gene D. ~5-6, Caleb M. ~5-6, Andrew R. ~15, Jerome S. ~21 failures before success. Server-side confirmed clean every time. Pure client-side JS state bug.

---

### SOIL™ Component Status (2026-03-21)

| Component | Status |
|-----------|--------|
| `session_create.php` | ✅ Working |
| `session_read.php` | ✅ Working |
| `session_update.php` | ✅ Working |
| Customer page `/intake-c/` | ✅ Working end-to-end |
| Basic Auth gate | ✅ Working |
| PHP excluded from auth | ✅ Working |
| Self-hosted fonts | ✅ Deployed |
| Sessions dir locked | ✅ `Require all denied` |
| Operator Step 2→3 | ⚠️ ACTIVE BUG — SESSION state race |
| QR URL in operator page | ⚠️ Verify not hardcoded to `/intake/customer/` — must be `/intake-c/` |
| Photo upload end-to-end | ⚠️ Not yet tested with real blade photo |
| Color-coded day system | 🔲 JS map ready, not wired |
| Wall of Edge™ bridge | 🔲 Future thread |

---

### Webserver Note

Deployment target runs **Apache** (switched from LiteSpeed). Apache 2.4 `.htaccess` inheritance rules apply. Do not assume LiteSpeed behavior.

---

### SOIL™ Operating Principles

- Never touch `/start/` during intake work — Astro landing page deploys independently
- Session files are ephemeral — Wall of Edge™ is the persistence layer
- Customer URL is `/intake-c/?s=[token]` — full token, not short code
- Photo storage is flat file — base64 never lives in session JSON
