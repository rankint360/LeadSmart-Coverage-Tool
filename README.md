# LeadSmart Coverage Tool

A byte-for-byte clone of the LeadSmart Coverage Intelligence tool
(lead payouts by niche, city & ZIP), deployed live for public use.

**Live site:** https://leadsmart-live-coverage.netlify.app/
(deployed from this repo's `main` branch to Netlify, team `rankintelligence`;
auto-deploys on every push)

## START HERE — for any AI or developer continuing this project

1. **Read `STATUS.md`** — what is done, what is pending, and what is waiting
   on the owner. It is refreshed after every work step.
2. **Read `RECORD_REGISTER.md`** — the append-only history (entries R-001…)
   of everything that was done and why. Add the next entry and refresh
   `STATUS.md` after each build step, then push both files.
3. Key facts a newcomer needs:
   - The site is fully static: `index.html` + `data.js` + 412 precomputed
     JSON files under `api/`. The copies in this repo are a frozen
     2026-11-12 snapshot and act as fallback only.
   - Live data does NOT come from those copies: `netlify.toml` force-proxies
     every `/api/*` and `/data.js` request to
     `leadsmart-coverage.netlify.app`, which itself syncs several times a
     day from LeadSmart's affiliate bid feed
     (affiliate-bid-coverage.leadsmartinc.com) with city population metrics.
     So the live site always shows the current dataset with no re-uploads.
   - A plan to replace that proxy with our own sync directly from
     LeadSmart's database (Supabase `offer_bids`/`verticals`) is scoped and
     feasibility-verified — see Register entry R-016 and `STATUS.md`.
   - There are no secrets in this repo. Do not add any.

## Layout

- `index.html`, `data.js`, `favicon.svg`, `robots.txt`, `sitemap.xml` — the tool itself
- `api/` — snapshot data files (fallback; live traffic is proxied, see above)
- `netlify.toml` — Netlify config incl. the live-data proxy rules
- `STATUS.md`, `RECORD_REGISTER.md` — project status and history (start above)
