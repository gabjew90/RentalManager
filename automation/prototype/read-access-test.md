# Read-access test: Furnished Finder (2026-07-04)

Question: can the agent, from this environment, actually read Furnished Finder data today?

## Results

| Path | Test | Result |
|---|---|---|
| Public FF pages (listings, search) | `curl https://www.furnishedfinder.com/` via session proxy | ❌ **403 at the environment gateway** — `connect_rejected: policy denial` for `www.furnishedfinder.com:443`. Blocked by this environment's network policy, **not** by Furnished Finder |
| Same, via WebFetch | `WebFetch(furnishedfinder.com)` | ❌ Same 403 — same egress policy |
| Gmail (FF lead emails) | Gmail connector tool availability | ❌ Connector exists but is **not authorized** — no Gmail tools available until OAuth is completed by the operator |
| FF landlord dashboard (behind login) | — | ⏸️ Untestable until the network policy allows the domain; additionally requires operator credentials (store as environment secrets, never in chat/repo) |

## What this means

Nothing about FF itself blocked us — both failures are **operator-side switches**:

1. **Network policy** — in the Claude Code environment settings (claude.ai/code → this environment → network/permissions), allow `furnishedfinder.com` (and `www.furnishedfinder.com`), or select a broader egress policy. Unblocks: public listing reads, and later browser-based dashboard access.
2. **Gmail connector** — claude.ai → Settings → Connectors → authorize Gmail. Unblocks: lead-email ingest, the primary event source for the whole lead-followup workflow.

## Re-test plan (once flipped)

1. Fetch both of our public listing pages → confirm content is server-rendered and parseable (listing title, availability date, rate shown).
2. Gmail: search `from:furnishedfinder.com` → parse the most recent real lead email against the workflow's intake spec → compare with the fixture-based prototype.
3. Only then evaluate dashboard reads (Playwright + credentials as env secrets) — needed for anything FF doesn't email out.
