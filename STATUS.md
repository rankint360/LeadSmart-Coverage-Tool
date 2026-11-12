# STATUS — LeadSmart Coverage Intelligence (Clone)

Last updated: 2026-11-12 11:58 PST (after Register entry R-011)
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
| Daily auto-update | ⏳ Not built | Designed and explained to user; waiting on user's go-ahead (offered 2026-11-12 11:01 PST). |
| GitHub push | ⚠️ 415 of 419 files pushed | Repo: [rankint360/LeadSmart-Coverage-Tool](https://github.com/rankint360/LeadSmart-Coverage-Tool), branch `main`, 18 Muse commits (R-011). Missing: `api/bundles/_all__2.json` to `_all__5.json` (5–8 MB each) — they exceed the GitHub connector's per-request size limit (HTTP 413). User can upload those 4 via GitHub web UI into `api/bundles/`. The user's pre-existing `leadsmart_coverage_tool.zip` in the repo was left untouched. |
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
- **User action pending (GitHub):** upload the 4 oversized nationwide bundle files (`api/bundles/_all__2.json` … `_all__5.json`) via GitHub web UI, or provide another route — see R-011. Until then, the repo copy's nationwide Bundles view (all states, min niches 2–5) has no data; all state-level bundles work.

## Next steps (in order)

1. **Daily auto-update for the clone** — *needs user's go-ahead.*
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
