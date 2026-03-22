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

### RESOLVED — Operator Step 2→3 Failure (2026-03-21)

**Symptom:** Tapping "Start Session →" returned to Step 1 with no network request. Appeared as a JS state bug. Affected every operator; frequency increased with faster tapping.

**Root cause (confirmed):** Layout mis-tap, not a JS race. The `← Back` button was positioned immediately below "Start Session →" with only `margin-top: 10px` of separation. Both buttons are full-width (`max-width: 320px`). `← Back` has `pointer-events: all` unconditionally (no `ready` class required). Operators tapping the bottom edge of "Start Session →" were hitting `← Back` instead, which called `backStep1()` → `show('screenPrefill')` — returning to Step 1 silently.

The SESSION state and `goStep3()` guard were never the issue. `goStep3()` was not being called at all on failed attempts.

**Fix deployed:**
1. Moved `← Back` to the **top** of screen 2 (above step dots and heading) — no longer adjacent to the CTA
2. Added `margin-top: 48px` to `.cta` — increases separation between color chips and Start Session
3. Added `padding-bottom: 24px` + `border-bottom` hairline to `.color-chips` — visual separator reinforcing the zone boundary

**Field data:** Paul Bragg ~3 attempts, Gene D. ~5-6, Caleb M. ~5-6, Andrew R. ~15, Jerome S. ~21 failures. Server-side always clean. Pure layout/touch-target issue.

---

### RESOLVED — Peach Chip / Color Selection Failure (2026-03-21)

**Symptom:** Tapping any color chip (most visibly Peach, the lone chip in row 2) returned the operator silently to Step 1. No console output. No network request. `selectColor()` never fired.

**Root cause 1 — photoCapture intercept:** `.photo-capture` div had `onclick` on the entire element (`aspect-ratio: 4/3`, ~240px tall). It sat directly above the color chips. Tapping Peach landed on the bottom edge of `photoCapture`, triggering `photoFile.click()` → camera/file picker launch → browser scroll/blur cycle that visually reset the page. Not `backStep1()` — the file input trigger caused the disruption.

**Root cause 2 — hidden screen tap bleed (deeper issue):** `.screen.hidden` used `opacity: 0; pointer-events: none` but screens are `position: fixed; inset: 0` — all stacked in the same viewport simultaneously. `screenQR` (DOM order 3) sits on top of `screenPhoto` (DOM order 2). `screenQR`'s `New Intake →` button has `class="cta ready"`, and `.cta.ready { pointer-events: all }` overrode the inherited `pointer-events: none` from `.screen.hidden`. Tapping Peach (bottom of screen 2) hit the invisible `New Intake →` button from screen 3, calling `newSession()` → Step 1. Same silent symptom.

**Fix deployed:**
1. `photoCapture` div made non-interactive (`pointer-events: none`). Replaced whole-div `onclick` with a scoped `<button type="button" class="photo-capture-btn">` inside it. `Retake` span given its own `pointer-events: auto`.
2. `.screen.hidden` changed to `display: none` — fully removes hidden screens from layout and tap stack. No child `pointer-events` override can punch through.
3. Entry animation moved to `@keyframes screenIn` on `.screen:not(.hidden)` to preserve the fade-in/slide-up transition.

**Key lesson:** `pointer-events: none` on a parent does NOT suppress children that explicitly set `pointer-events: auto` or `pointer-events: all`. Only `display: none` or `visibility: hidden` fully cuts off descendants.

---

### SOIL™ Component Status (2026-03-21, updated 2026-03-21)

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
| Operator Step 2→3 | ✅ Fixed — layout mis-tap (← Back proximity) |
| Color chip selection | ✅ Fixed — photoCapture intercept + hidden screen tap bleed |
| `newSession()` DOM cleanup | ✅ Fixed — QR screen elements + `--session-color` CSS var now fully reset |
| Hidden screen tap isolation | ✅ Fixed — `display: none` on `.screen.hidden` |
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
## EST™ Instance Protocol — TempleForge™ Canon

*The SomeBody™ Company · BiMKA™ · TiO™ Standard*
*Locked: 2026-03-22*

### Two instances. One job. Clear lanes.

| Instance | Role | Does NOT |
|---|---|---|
| **Claude.ai (chat)** | Strategy, decisions, brand judgment, QA review via browser, CTS™/CQT™ artifacts, accumulated project context | Touch the server directly |
| **Claude Code (WSL terminal)** | File reads, patches, SSH/SCP deploys, git operations, server-side execution | Hold long-form context or make architectural decisions |

### The Rule

Anything that touches the server belongs to Claude Code.
Anything that requires accumulated project knowledge, brand judgment, or architectural decisions belongs to the chat instance.
Never reverse this without a documented reason.

### The Handoff Pattern

1. Chat instance identifies what needs to change and reasons through the patch logic
2. Claude Code pulls the live file, applies the patch, deploys, confirms
3. Chat instance QAs via browser tools and closes the loop
4. CTS™ artifact seals the session — committed to git by Claude Code

### Token Discipline

Claude Code handles execution precisely because it does not carry conversation weight. Routing execution tasks through the chat instance wastes tokens and slows the loop. TiO™ demands the right tool for the right surface. When in doubt: **think here, execute there.**

### On Assets — Non-Negotiable

No synthetic placeholders — ever. If a brand asset exists on the server (PNG, SVG, favicon, font), use it. A drawn substitute, an emoji stand-in, or a CSS approximation is not a brand asset and must never appear in any deliverable. The orb is the orb. The font is the font. Brands that build active, viral followings never leave placeholders in the chain.

### On Scripts and Manual Handoffs

Writing a deploy script and asking the operator to run it is the lesser path when Claude Code is available and active. This pattern is only acceptable when Claude Code is not running. If Claude Code is open, Claude Code executes. The chat instance does not write relay scripts for the operator to run manually — that is an open loop, a tax on the operator, and a violation of TiO™.

### Login Refresh

Claude Code sessions may require periodic `/login` refresh due to OAuth token expiry. This is expected behavior — not a failure. The operator re-authenticates and execution resumes. The cost is negligible relative to the efficiency gained.
