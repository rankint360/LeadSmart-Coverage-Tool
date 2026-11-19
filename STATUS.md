# STATUS — LeadSmart Coverage Intelligence (Clone)

Last updated: 2026-11-18 23:52 PST (after Register entry R-016)
Full history: see `RECORD_REGISTER.md` in this folder. This file is the current state only — it is rewritten after each build step, and the Register is appended at the same time.

## What this tool is

A byte-for-byte clone of https://leadsmart-coverage.netlify.app/ — a static tool that shows what home-service leads pay by niche, city, and ZIP code, with coverage and bundle (stackable market) views.

- Original (upstream reference): https://leadsmart-coverage.netlify.app/
- Data source (LeadSmart's own tool): https://affiliate-bid-coverage.leadsmartinc.com/
- Population / county / geo enrichment: SimpleMaps US Cities dataset (CC BY 4.0) — not from LeadSmart

## Current state

| Component | State | Notes |
|---|---|---|
| Clone (files) | ✅ Done, verified | 417 files, byte-identical to original. Folder: `~/workspace/your_files/leadsmart-coverage-clone/` · Zip: `~/workspace/your_files/leadsmart-coverage-clone.zip` (~10 MB) |
| Dataset in clone | 📌 Frozen snapshot | 2026-11-12 dataset only: 268,401 coverage rows, 28,372 cities, synced 2026-11-12T18:18:20Z, fingerprint `2026-11-12T16:51:06.283257+00:00\|268401\|724f525c81`, `DB_VERSION 20261112-181854`. Does **not** refresh by itself. |
| In-chat preview | ✅ Fixed, verified by Muse | Space slug: `leadsmart-coverage-intelligence`. Data loading repaired 2026-11-12 (see Known issues). Verified in Chromium under the real preview conditions: header stats, Top markets, Tulsa (By city / ZIP), By state, and Bundles (21,237 markets) all populate. |
| Daily auto-update | 🔄 Replaced by live proxy (R-013) | Instead of scheduled re-uploads, the deployment proxies `/api/*` and `/data.js` live from the upstream site (`netlify.toml` in repo root), so the public site always shows the current dataset. Upstream was already on 2026-11-18 data (258,109 rows) at check time, vs the repo snapshot's 2026-11-12 data. |
| Public deployment | ✅ Live and verified | **https://leadsmart-live-coverage.netlify.app/** — Netlify site in the user's `rankintelligence` team, deployed from repo `main` @ `b835ca4` on 2026-11-18 23:31 PST, public, auto-deploys on push (R-014). Renamed by the user from Netlify's random `statuesque-bunny-03cabc` name to `leadsmart-live-coverage` on 2026-11-18 (R-015; old name now 404s). Verified serving the live dataset (data_date 2026-11-18, 258,109 rows) via the R-013 proxy — not the repo snapshot. |
| How updates reach the live site | 🔄 Two-hop chain, no work on our side | (1) LeadSmart refreshes its bid database behind affiliate-bid-coverage.leadsmartinc.com. (2) leadsmart-coverage.netlify.app's backend checks that feed several times a day and, when its fingerprint changes, rebuilds its 412 JSONs (with population data) and redeploys itself. (3) Our site's `netlify.toml` proxies every `/api/*` and `/data.js` request to that site at request time — so our visitors get hop 2's newest dataset the moment it publishes, with no copies, cron jobs, or redeploys on our side. Dependency: if the upstream site goes down or changes layout, our data follows it (fallback snapshot in repo is 2026-11-12). |
| GitHub push | ✅ Complete — 419 of 419 files | Repo: [rankint360/LeadSmart-Coverage-Tool](https://github.com/rankint360/LeadSmart-Coverage-Tool), branch `main`. Muse pushed 415 files in 19 commits (R-011); the 4 oversized nationwide bundle files were uploaded by the user via GitHub web UI on 2026-11-18 (commit `9a70349`) and verified by Muse — full repo tree matches the clone with zero size mismatches (R-012). The user's pre-existing `leadsmart_coverage_tool.zip` in the repo was left untouched. |
| Project records | ✅ Created 2026-11-12 | `RECORD_REGISTER.md` (R-001–R-008) + this `STATUS.md`, in `~/workspace/your_files/leadsmart-coverage-project/` — kept outside the clone folder so the clone stays unchanged. |

## Done

1. Cloned the original site without any changes — all 417 files including the 412 `/api/` data files (R-002, R-003).
2. Verified the clone: core-file SHA-256 hashes match the original, all JSON valid, local serve test passed, and the tool's own data logic produced correct results from the local files alone (Tulsa, ZIP 74101, Roofing, TX bundles) (R-004).
3. Packaged and delivered the clone as a zip (R-004).
4. Built an in-chat preview of the clone (R-005).
5. Diagnosed and fixed the preview's missing-data failure — blocked data fetches in the preview frame plus an unguarded storage write — and re-verified all main flows (R-006).
6. Traced and documented how the original gets its daily data from LeadSmart (scrape → Supabase `offer_bids` → fingerprint-checked sync → SimpleMaps enrichment → precompute → Netlify redeploy) (R-007).
7. Created this STATUS.md and the Record Register, backfilled with the full history (R-008).

## In progress / awaiting

- **User confirmation pending:** whether the repaired preview now shows the live data for the user (fix verified by Muse on 2026-11-12 10:55 PST; user has not yet confirmed).
- **User decision pending:** whether to build the daily auto-update (see Next steps, item 1).
- **GitHub upload:** ✅ done — user uploaded the final 4 bundle files on 2026-11-18 and Muse verified all 419 files in the repo (R-012). Nothing outstanding on the GitHub push.

## Own sync direct from LeadSmart (R-016) — feasibility ✅ verified, awaiting user's go-ahead

LeadSmart's affiliate tool reads a Supabase database with a public anon key shipped in its own JS; Muse read `verticals` and `offer_bids` live with it (freshest rows 2026-11-18T14:02 UTC, matching upstream). Build = Extractor (fingerprinted, keyset-paginated — whole-table sorts time out on their DB) + SimpleMaps enrichment + Precompute replicating the 412 files (validated by diffing against upstream's published output for the same fingerprint) + GitHub Actions publisher committing to `main` (Netlify auto-deploys; proxy then removable or kept as fallback). Risks: LeadSmart could rotate the key / tighten access; data is LeadSmart's (affiliate terms apply).

## Next steps (in order)

0. ✅ **Deploy to Netlify and verify — DONE (R-014, 2026-11-18).** Live at https://leadsmart-live-coverage.netlify.app/ (renamed by user, R-015) and verified serving the current dataset. Optional follow-up if the user wants it: attach a custom domain in Netlify.
1. **Daily auto-update for the clone (superseded by the live proxy above; kept for reference)** — *needs user's go-ahead.*
   Build a scheduled job that:
   - Polls the original's `api/meta.json` / `data.js` version stamp several times a day (or at least daily) and compares the source fingerprint to the one recorded above.
   - When it changes: re-downloads the refreshed files (all 417, same method as R-003), re-verifies hashes/JSON/flows as in R-004, and re-zips the clone.
   - Refreshes the in-chat preview with the new dataset (keeping the R-006 data-loading fix, so the preview keeps working in its frame).
   - Appends a Register entry with the new `data_date`, fingerprint, and row count, and updates this STATUS.md.
   - Note: this tracks the Netlify site's published dataset. A direct-from-LeadSmart version (polling LeadSmart's own feed/database and doing the SimpleMaps enrichment and precompute ourselves, like the original's `sync_supabase.py` + `precompute.py`) is a larger build and would be scoped separately if the user prefers not to depend on the Netlify site.
2. **Confirm preview with user** — ask the user to open the repaired preview and confirm data shows; if any flow still fails for them, log it in the Register and repair.
3. **(Optional, only if user asks)** Point the clone at its own domain: update canonical / `og:url` tags (currently pointing to the original, deliberately left unchanged) and deploy the clone folder to Netlify or another static host.

## Known issues / notes

- The clone's data is a **snapshot, not a live feed** — payouts shown are as of 2026-11-12 until the auto-update (Next step 1) exists.
- In the in-chat preview only: the browser console logs the first-attempt data fetch being blocked before the fallback loads the data anyway; copy-to-clipboard and CSV-download buttons depend on frame permissions the page cannot grant itself. Neither affects the downloaded clone when hosted normally.
- On phone widths, two inner areas of the original design scroll sideways (main tabs row, payout table). That is the original site's own layout and was deliberately left unchanged.
- The original site states it is an independent research tool, not operated by or affiliated with LeadSmart — that disclaimer is part of the cloned page and remains in place.

## Update rule

After **each** build step on this tool: append the step to `RECORD_REGISTER.md` (next R-number) and refresh this file's "Last updated", tables, and lists **before** telling the user the step is done.
