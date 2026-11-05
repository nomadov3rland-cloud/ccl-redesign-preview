# Publishing this site — read before you push

**Publish source of truth: the Omarchy clone** (`~/columbia-county-live/site`
on Travis's computer). The fetcher writes fresh snapshots into that clone and
its runners push from there. That copy is what "live" means.

**This VM workspace copy (`files/site/`) is the code master, not a data
source.** Its `data/` folder is a mirror that drifts stale between syncs —
on 2026-11-04 it held a bogus `key_needed` ships.json and ~29-hour-old
snapshots while the live site was healthy, and a wrong-copy publish already
regressed live data once.

Rules for anyone building or publishing here:

1. Edit code in this master; publish code via the established routes
   (connector push-files, or copy into the Omarchy clone and push under the
   shared `publish.lock` flock — see the sync record in
   `files/site.muse-github-snapshot-sync.json` and `fetcher/README.md`).
2. **Never publish `data/*.json` from this master without syncing it from
   the Omarchy clone first**, then verifying every file's `fetched_at`
   (fast lane: minutes old; full lane: < ~35 min) and `status` (no
   `key_needed` surprises). If in doubt, publish code-only paths and let
   the fetcher own data — it republishes everything within 30 minutes.
3. Bump the service-worker version in `sw.js` on every code publish and run
   the execution harness to ALL PASS before pushing.
4. A watchdog (`hidden_files/watchdog/`) checks the live files' timestamps
   every 15 minutes and alarms into the main chat if they go stale — a
   post-publish alarm means check your publish set first.

## Build checklist — every site change, before publish

1. Service-worker version bumped in `sw.js` (one bump per publish).
2. Execution harness (`hidden_files/iterations/ccl-harness-redesign.js`)
   run against the master — ALL PASS, not just "no worse".
3. **Spanish strings for dynamic content:** any user-visible string the
   JavaScript renders (status labels, verdicts, pills, banners, button
   text it rewrites) must render in Spanish in ES mode, not just the
   static `data-i18n` chrome — either via the `LANG_ES` dictionary for
   static text or a per-module EN/ES string table + a re-render hook on
   the language toggle (pattern: the fish pick card in `langSet`, the
   schools board wrap in the WS-1 module). Content quoted from an
   official source (alerts, listings, a district's own note) stays in
   the source's language, and the page already says so.
4. No privacy-reassurance boilerplate copy ("nothing leaves your
   phone…") — Travis's standing rule. Genuine data-honesty labels
   (sources, timestamps, "modeled estimate") stay.
5. New cards: honest dormant/fallback state in static HTML first, JS
   enhancement second; data files the fetcher doesn't own (e.g.
   `data/schools.json`) are published code-side only, never mixed into
   a fetcher-data publish without the freshness check above.
6. Publish, then verify LIVE (cache-busted fetch), then log to
   `hidden_files/iterations/ITERATION-LOG.md`.
