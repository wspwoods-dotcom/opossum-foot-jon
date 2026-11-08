# Opossum Foot — Developer Handoff

**Product:** Opossum Foot (with an O — "We have opossums in Iowa, not possums").
A phone-first trap log for trappers. Local-first PWA: no accounts, no cloud,
no analytics, no backend. All user data stays on the trapper's phone.

**Owner:** Tanner Scheuermann / Opossum Foot LLC (Iowa entity #888474, Certificate of Organization filed 9/24/2026; EIN assigned 9/25/2026).
**Status at handoff:** closed beta, 6 official testers + 1 adjacent with access (see `beta-tester-roster.md`). Current build:
`beta 0.1 · build 2026-09-25bh`, cache `opossum-foot-shell-v37` (see "Versioning" below).

---

## 1. Stack — there is no build step

Plain static site. No npm, no bundler, no framework. Upload the folder
contents and it runs.

- `index.html` — every screen: splash, state picker, map, history, totals,
  seasons, license wallet, settings; bottom sheets for sets / catch logging.
- `css/styles.css` — mobile-first, big touch targets, field-readable theme.
  Page body is pinned (`overflow: hidden`); only inner panes scroll — this
  keeps the bottom tab bar from dragging on iOS Safari.
- `js/app.js` — all application logic (sets, logs, seasons, voice, CSV/JSON
  export, PWA wiring). `'use strict'`, ES5-style. No `console.log` in ship
  code.
- `js/season-data.js` — all 50 states' 2026–27 season data embedded as a
  plain script so the app works from `file://` with zero fetch/CORS issues.
  Generated from `data/*.json` (see §5). Only the user's selected state is
  used at runtime.
- `js/guide-content.js` — generated from `FOR-TRAPPERS.md` (see §6). Do not
  hand-edit; regenerate.
- `data/*.json` — 50 source season-data files, schema v1.1.
- `assets/` — logo + generated PWA icons (192/512/180/32) + species art.
- `manifest.json`, `sw.js` — installability + offline app-shell/tile caching.
- `tools/build-guide.js` — `node tools/build-guide.js` regenerates
  `js/guide-content.js` from `FOR-TRAPPERS.md`. Run it before every deploy
  that touches the guide.
- `FOR-TRAPPERS.md` — the in-app Trapper's Guide source (Settings → Help &
  legal). **Accuracy audit incomplete** — several statements were baked in
  before verification (license-photo labeling/offline availability, whether
  sprung/empty logs affect catch-per-trap-night, "no account, no cloud"
  wording vs. future store plans). Verify against the running app before
  editing.
- `LEGAL-NOTICE.md` — the in-app Legal notices source. **Not lawyer-reviewed.**
  Open question: copyright ownership wording names "Opossum Foot LLC" in-app,
  while the federal copyright filing prep names Tanner Scheuermann personally.
  Do not change this wording without the owner's decision + lawyer review.

## 2. Run it locally

```sh
cd possum-foot-app
python3 -m http.server 8000   # or any static server
# open http://localhost:8000
```

`file://` also works for smoke-testing screens (service worker is skipped
there — expected; offline app-shell caching needs http/https).

## 3. Data model (on-device only)

| Store | Key / DB | Contents |
|---|---|---|
| localStorage | `opossumfoot.v1` | Everything JSON: sets, catch logs, settings, trap lines |
| localStorage | `opossumFootBackupPreLines` | One-time pre-migration backup (Trap Lines feature) |
| IndexedDB | `opossumfoot` (v1) | Blobs: license photos, voice memos |
| localStorage | `opossum-foot-sets-*`, `opossum-foot-catches-*` | Legacy per-device keys (older builds) |

Export paths the app already supports: CSV and JSON (Settings). There is no
server, so "sync" = the user exporting a file.

## 4. Versioning — bump two places every deploy

Build identity is a human-readable tag, e.g. `beta 0.1 · build 2026-09-24bg`
(date + sequential two-letter suffix):

1. `js/app.js` → `var APP_VERSION = '...'` (shown in Settings → About; this is
   what the owner checks on his phone to confirm a deploy).
2. `sw.js` → `var SHELL_CACHE = 'opossum-foot-shell-vNN'` (bump the number;
   this forces phones to fetch the new shell instead of serving stale cache).

If either is stale, testers will run old code from cache. Both must change
together, every deploy.

## 5. Season data

- Source of truth: `data/<state>-2026-27.json`, schema v1.1.
- 46 states final on 2026–27 rules; **Connecticut, Mississippi, North
  Carolina, Vermont are provisional** (`"provisional": true` in `js/app.js`
  `STATES`) — the app must not be called launch-ready until those four are
  re-verified against the official state sources.
- Season info is **reminder-only**: the app never claims to decide whether
  trapping is legal, and the UI must keep that framing.
- Tile layers: Esri keyless street map (switched from CARTO Sept 2026 after
  CARTO tiles showed an API-key watermark). Runtime network use is only: map
  tiles (cartocdn/arcgisonline/opentopomap) and keyless
  reverse-geocode (bigdatacloud, fallback nominatim). No user data is ever
  transmitted.

## 6. Deploy (GitHub Pages)

Live repo: `wspwoods-dotcom/opossum-foot` (branch deploy, files at repo root).
Live URL: https://wspwoods-dotcom.github.io/opossum-foot/

The deploy is performed by uploading changed files through the GitHub web UI.
**Freeze rule:** finish ALL local edits before starting the upload session —
upload grants are digested when the session starts, so a later local edit
invalidates that file's grant and it silently fails to deploy.

Release protocol (owner's rule, do not skip):
1. Deploy, then verify the live `app.js`/`sw.js` contain the new build tag
   and cache name.
2. Owner verifies on his iPhone first (Settings → About shows the new build).
3. Only then do the beta testers move — all together, never piecemeal.
   **Routine build moves: testers NEVER delete the home-screen icon.**
   The installed app picks up the new build on launch (fully close and
   reopen if it shows the old build letter). Deleting the iPhone icon wipes
   the phone's saved data — lesson learned 2026-09-25 when a tester lost
   all records following delete-and-reinstall instructions.

## 7. Beta distribution policy (owner-set, Sept 2026)

- 6–8 testers max until store testing tracks exist (currently 6 official +
  1 adjacent — nearly at cap). Demo widely, distribute narrowly. Never send
  the beta URL unsolicited.
- At the phase change (sponsor model / wider release): testers run
  Settings → Backup → "Back up all data (JSON)" first, then delete the app,
  receive a fresh link on a new repo, old repo killed same day — once, not
  routinely.

## 8. Known backlog (not started at handoff)

Full queue with specs: `feature-queue.md` (owner's rule — ideas are queued
and refined first, then built in Tanner-authorized batches; never build
suggestions one at a time). Headlines:
- Q1: Regenerated Trapper's Guide (`tools/build-guide.js` output) — built,
  **not deployed**; blocked on the Q2 factual audit.
- Q2: Trapper's Guide factual audit (see §1 FOR-TRAPPERS.md note).
- Q4: GPS follow watchdog — restart the position watch after ~30–45 s
  without a fresh fix / on page re-visible; show "Reacquiring GPS…";
  show "Follow off — map dragged" when a drag disables follow; surface
  stale/weak fixes instead of silently discarding them.
- Q5: Scorecard becomes its own bottom tab (out of History); likely fix —
  stamp lure/bait onto each catch record at log time so later set edits
  can't reassign old catches.
- Q6: History tab declutter — filters are toggle chips only (By date /
  By species view + disposition chips); full report/export filters move to
  Settings beside CSV export with **independent** filter state.
- Accuracy audit of the baked-in Trapper's Guide (see §1).
- Multi-license wallet: several license photos tagged by owner (storage only —
  never allocate catches or legal limits among family members).
- Google Maps import of 22 saved sets (duplicate + required-field behavior
  unresolved).
- App Store / Google Play path: developer accounts, native wrapper vs.
  rebuild, review, privacy disclosures, TestFlight/Play tracks — owner wants a
  plain-language cost/options briefing before committing.
- Re-verify CT, MS, NC, VT season data (see §5).
- Per-set cellular-camera link idea; map-layer status toggles.

## 9. Standing product rules (do not break)

- Name is **Opossum Foot**; business is **Opossum Foot LLC**.
- Say "lure and bait makers", never "lure nerds". Promotion is
  audience-building, not selling. No public pricing. Never badmouth IronLine.
  Never claim it is the first trapping app.
- Voice entry uses browser speech recognition — never claim it always works
  offline.
- Family/kid content: draft → owner approves → owner sends. Nothing direct.
