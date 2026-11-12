# Record Register — LeadSmart Coverage Intelligence (Clone)

Project: Clone of LeadSmart Coverage Intelligence
Original: https://leadsmart-coverage.netlify.app/
Upstream source: https://affiliate-bid-coverage.leadsmartinc.com/ (LeadSmart's own affiliate bid coverage tool)
Register started: Thursday, 2026-11-12 (America/Los_Angeles)
Maintained by: Muse, for the user

## How to use this register

- This is the permanent, chronological record of everything done on this tool: requests, build steps, verification, failures, fixes, and decisions.
- Add one entry per build step, in order, as the step completes — never rewrite or delete earlier entries. If an earlier entry turns out to be wrong, add a new correction entry that references it.
- Every entry records: date/time, what was asked or attempted, what was actually done, evidence/verification, and the resulting state.
- `STATUS.md` in this same folder is the companion file: it always shows the current state only (done / in progress / next / blocked). Update `STATUS.md` at the same time as every entry added here.
- Rule set by the user on 2026-11-12: **update both files after each build step.**

## Entries

### R-001 — 2026-11-12 10:20 PST — Clone requested
- Request (user): "Clone this tool without any changes https://leadsmart-coverage.netlify.app/"
- Decision: Clone byte-for-byte, including all data files — not a visual re-creation. No design, copy, or data changes.

### R-002 — 2026-11-12 10:20–10:23 PST — Source site analysed
- Found the site is fully static, but not a single page. Structure:
  - `index.html` (117,191 bytes), `data.js` (19,697 bytes), `favicon.svg`, `robots.txt`, `sitemap.xml`
  - `data.js` is a client-side data layer: the browser never calls a live API; it fetches precomputed static JSON under `/api/`.
  - API files discovered from `data.js`: `meta.json`, `states.json`, `niche_summary.json`, `top_markets.json`, `cities.json`, `zipmap.json`, plus per-state shards, per-niche files, state summaries, and bundle files (see R-003).
- Dataset identity at clone time (from `api/meta.json`):
  - `data_date`: 2026-11-12
  - `data_updated_at`: 2026-11-12T16:51:06Z
  - `synced_at`: 2026-11-12T18:18:20Z
  - `source_fingerprint`: `2026-11-12T16:51:06.283257+00:00|268401|724f525c81`
  - `coverage_rows` / `source_top_bids`: 268,401
  - Totals: 28,372 cities, 40,218 ZIPs (meta total), 20 niches listed / 22 niche + payout-type combos
  - `data.js` version stamp (`DB_VERSION`): `20261112-181854`

### R-003 — 2026-11-12 10:21–10:23 PST — Full clone downloaded
- Downloaded all 417 files verbatim to `~/workspace/your_files/leadsmart-coverage-clone/`:
  - 5 core files: `index.html`, `data.js`, `favicon.svg`, `robots.txt`, `sitemap.xml`
  - 412 JSON files under `api/`: base files (6), `state/<STATE>.json` and `state_summary/<STATE>.json` for all 51 states (incl. DC), `niche/<slug>__<CPL|Call>.json` and `niche_states/<slug>__<CPL|Call>.json` for all 22 niche/payout combos, `bundles/<STATE|_all>__<2–6>.json` for every state plus nationwide.
  - Download result: 406 files in the bulk pass + 11 already present = 417 total, 0 failures, ~86 MB uncompressed.
- Correction during this step: the first copy of `index.html` fetched via Python was missing a Netlify-injected HTML comment (326 bytes) present in the curl copy. Replaced with the curl copy so the clone is truly byte-identical.

### R-004 — 2026-11-12 10:23 PST — Clone verified and packaged
- SHA-256 hashes of `index.html`, `data.js`, `favicon.svg`, `robots.txt`, `sitemap.xml` match the original site exactly.
- All 412 JSON files parse as valid JSON, 0 bad files.
- Served the clone locally (`python3 -m http.server`) — all key files returned HTTP 200.
- Ran the clone's own `data.js` logic in Node against the local files only:
  - meta: 20 niches, 40,218 ZIPs
  - niche summary: 22 rows, top = Biohazard, median $224.50
  - Tulsa, OK coverage: 10 niches; suggest "Tulsa" resolves correctly
  - Roofing (Call) niche loads; TX bundles (min 3 niches): 1,022 markets
  - ZIP 74101 resolves to Tulsa with 10 niches
- Packaged as `~/workspace/your_files/leadsmart-coverage-clone.zip` (~10 MB compressed, 417 files).
- Noted (no change made, per instruction): canonical / `og:url` tags in `index.html` still point to leadsmart-coverage.netlify.app, exactly as in the original.
- Deliverables delivered to user: clone folder + zip.

### R-005 — 2026-11-12 10:24–10:29 PST — In-chat preview built
- Request (user): "Can you preview this here?"
- Built a hosted static preview (Space) in Muse: name "LeadSmart Coverage Intelligence", slug `leadsmart-coverage-intelligence`.
- Builder audit at the time reported all flows working from the cloned local data (By city Tulsa, ZIP 74101, By niche Water Damage, By state AZ, Bundles 21,237 markets) and desktop/mobile screenshots matching the original. Only mechanical adaptations: `data.js` inlined and its data path / favicon pointed at the preview's assets folder.
- Status at delivery: preview presented to the user in chat.
- **Later superseded by R-006: the audit's test conditions differed from the real in-chat frame, and the preview did not actually show data for the user.**

### R-006 — 2026-11-12 10:45–10:55 PST — User reported preview shows no live data; root cause found and fixed
- Report (user): "It looks same but it does not work as https://leadsmart-coverage.netlify.app/ works" — clarified: "It does not show live data which is (https://leadsmart-coverage.netlify.app/) showing."
- Root cause (reproduced in Chromium under the real preview conditions — sandboxed cross-origin iframe, opaque origin):
  1. Every data `fetch()` to the preview's static `/api/` files was blocked by CORS (the static file route sends no CORS headers), so header stats, Top markets, By city, By state, and Bundles all stayed empty.
  2. An unguarded `localStorage` write in the "recent searches" helper threw inside the sandbox before search results could render.
- Fix applied to the preview only (the downloaded clone in R-003/R-004 was not altered):
  1. Generated script-tag twins (`assets/apidata/*.js`, byte-identical data) for all 412 datasets; `data.js` now falls back to script-tag loading when `fetch` fails for any reason. On normal hosting `fetch` remains the primary path, so behaviour there is unchanged from the original.
  2. Storage writes now use an in-memory map with `localStorage` guarded in try/catch.
- Verification after fix (same sandboxed Chromium test): header shows "28,372 Cities · 20 Niches · Nov 12 Updated 2026" and "Data updated Nov 12, 2026 · today"; Top markets populated; Tulsa search shows its coverage table incl. Water Damage and bundle ZIP 74115; By state loads; Bundles shows 21,237 markets, Addison, TX first.
- Known residue (unchanged, outside the reported fault): in the sandboxed preview the browser console still logs the first-attempt fetch rejections before the fallback engages (data still arrives); copy-to-clipboard and CSV-download buttons depend on frame permissions the page itself cannot grant.

### R-007 — 2026-11-12 11:00–11:01 PST — Data update pipeline investigated and explained
- Question (user): how will data get updated daily from LeadSmart into this tool, noting LeadSmart's own tool lives at affiliate-bid-coverage.leadsmartinc.com and the Netlify site gives updated payout info with city population metrics.
- Findings:
  1. LeadSmart's own tool is a web app backed by Supabase (tables `offer_bids`, `verticals`). A scrape populates `offer_bids`: `zip, city, state, offer_key, offer_name, top_payout, last_updated`. On 2026-11-12 it reported "Latest scrape complete — Finished Nov 12, 8:51 AM", matching the Netlify dataset's `data_updated_at` of 16:51 UTC.
  2. The Netlify site is **not live-connected**: per its FAQ it checks LeadSmart's public affiliate bid feed several times a day and rebuilds only when the feed's fingerprint (`updated_at|row_count|hash`) changes. Sync script is named `sync_supabase.py` in the site's own code comments.
  3. Population metrics do **not** come from LeadSmart: city, county, population, density, lat/lng are matched from the SimpleMaps US Cities dataset (CC BY 4.0). ~20% of feed ZIPs (PO-box/admin) have no city match and show no population.
  4. A CI job (`app/precompute.py`, via `refresh.yml`) bakes all query results into the 412 static `/api/` JSON files, stamps a new `DB_VERSION` into `data.js`, and redeploys to Netlify.
- Conclusion given to user: **the clone is a frozen snapshot of 2026-11-12 and will not update by itself.** Making it update daily requires replicating steps 2–4 (poll → sync → enrich → precompute → redeploy/refresh).
- Offer made to user (not yet accepted at time of writing): set up a daily auto-update for the clone that only rebuilds when LeadSmart's data actually changes.

### R-008 — 2026-11-12 11:06 PST — This register and STATUS.md created
- Request (user): build a Record Register and STATUS.md for this tool, record what has been done and what is going to be done next, and update both after each build step.
- Done: created `RECORD_REGISTER.md` (this file) and `STATUS.md` in `~/workspace/your_files/leadsmart-coverage-project/`, backfilled with R-001 to R-008 from the actual work record.
- The project record files are kept **outside** the clone folder so the clone itself stays byte-for-byte unchanged, as originally requested.
- Standing rule from this entry onward: after every build step on this tool, append a new entry here and refresh `STATUS.md` before reporting the step as done.

### R-009 — 2026-11-12 11:08 PST — GitHub push requested; blocked on connection and target repo
- Request (user): "Can you push source/record files to GitHub?"
- Scope: push the clone source (417 files, ~86 MB, in `~/workspace/your_files/leadsmart-coverage-clone/`) plus the project record files (`RECORD_REGISTER.md`, `STATUS.md`).
- Checks done:
  - GitHub connector status: **not connected** (`gh` CLI is also not logged in, and direct SSH to github.com is blocked from this machine, so the connector is the only push route).
  - No GitHub repository for this tool exists in the user's records, and the clone folder is not a git repo — a target repository (existing or new) must be chosen after connecting.
  - Push method reviewed (GitHub connector snapshot push): per-push limits are 2,000 files / 16 MiB of content, so the ~86 MB source (59 MB of it `api/bundles/`) will be pushed in several batched commits, records + core files first. The snapshot method does not carry local git history/tags/file modes; acceptable for a source snapshot.
- Action taken: user given the GitHub connect link in chat on 2026-11-12.
- State: **waiting on user** to (1) connect GitHub, (2) select/create the repository Muse may access, and (3) confirm the repository name. Push will proceed immediately after, and the result (repo, branch, commit SHAs) will be recorded here.

### R-010 — 2026-11-12 11:24 PST — GitHub OAuth completed; repository selection pending
- Progress on R-009: user completed GitHub OAuth authorization successfully.
- GitHub App installation / repository access **not yet verified**: user sent the "Select repositories" installation link (`https://github.com/apps/meta-muse-ai/installations/new`) to choose which repositories Muse can access.
- Next action: once the user has selected/created the target repository and confirmed its name, verify access with a read, then push as planned in R-009 (records + core first, then the 412 data files in batched commits) and record repo, branch, and commit SHAs here.

## Open items carried into STATUS.md

See `STATUS.md` in this folder for the current state, next steps, and blockers. Summary at register creation:
- Daily auto-update: designed and explained, **not yet built** — waiting on user's go-ahead.
- Preview: fixed and verified by Muse; user confirmation that data now shows for them is still pending.
