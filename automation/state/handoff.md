# Session handoff — live status (updated 2026-07-18)

Branch: `claude/claude-md-docs-wmsn2b` · Mode: **supervised** · Scope: **MVP** (intake → evaluate → draft only; nothing auto-sends). Read this first, then `units.yaml`, `inquiries.yaml`, `meta.yaml`.

## Units
- **1B1B = Fremont** (39139 Argonaut Way #212). Economics complete incl. mortgage P/I split (principal ~$831, interest+escrow ~$1,030). FF listed $3,800. **Listing description NOT yet read** — pending FF network access; then populate `units-facts.yaml: fremont`.
- **2BR = Newark** (NewPark area; FF property 951920). Economics + break-even term ladder complete. FF listed $4,200. **Available Aug 10, 2026.** Real listing captured in `units-facts.yaml: newark`. **Mortgage P/I split still NEEDED** (operator gave the four fixed costs combined as $3,700) — blocks cash-vs-economic break-even.

## Active leads — 3-way race for the 2BR fall slot (only one wins)
All drafts sit in the operator's **Gmail Drafts** (not in the repo).
1. **Sebastien P.** (Jul 7) — Sep 1→Feb 28 (6mo). Preferred: firmest commitment; engineer end to ~Apr 30 = Sep→May anchor. Draft = FF paste ("[RM] PASTE INTO FF…") — no email on file, must paste into FF.
2. **Affan T.** (Jul 18) — Aug 17→Dec 31 then m2m. Local (works in Fremont, unit in adjacent Newark) = best fit, m2m = anchor potential. Aug 17 works (free Aug 10). **Draft v4 in Gmail is READY TO SEND** (subject "…furnished 2BR in Newark, minutes to Fremont"; listing URL furnishedfinder.com/property/951920_1 filled in — discard v1–v3). **SMS paste also in Drafts** ("[RM] SMS PASTE v3 — …310-728-5575", 296 chars, plain/direct tone per operator; discard v1/v2 SMS notes) — send with/after the email. **Tone note: operator wants natural, direct wording — no salesy phrasing.**
3. **Huzayfa J.** (Jul 3) — Sep 1→Dec 31. Cooling ("far from palo alto"). Now third. Objection-reply draft in Gmail (needs commute-time blank filled).

Recommendation: pursue Sebastien to a decision; Affan strong backup; treat Huzayfa as third. Block-before-yes — don't promise the unit to two parties.

## Pending operator actions
- Send/act on the drafts above (Sebastien via FF paste; Affan + Huzayfa via Gmail send).
- Affan's draft v4 has the listing URL filled in; only pet policy still unconfirmed (draft doesn't mention pets, so not blocking).
- Provide the 2BR mortgage principal/interest split.
- Decide the 2BR winner; say whether to keep pursuing Huzayfa.

## Owed / not yet built
- **Pricing worksheet** — requested/offered, NOT built. Would turn the per-unit cost stacks + per-stay overhead (gap + cleaning) + occupancy + fee model + term into a reusable calculator. Method: cost/occupied-month = base(fixed+elec) + overhead/N; overhead = 1-week-gap carry (fixed × 12/52) + cleaning/booking. Results already in each unit's `economics` block.
- **FF network access** — STILL blocked (re-checked 2026-07-18: proxy CONNECT to furnishedfinder.com → 403; egress policy fixed at container start). Public listing URL confirmed by operator: furnishedfinder.com/property/951920_1 (2BR). To verify listing text / finish `units-facts.yaml: fremont`, either allowlist furnishedfinder.com in the environment's network policy or paste the listing pages into chat.

## Infra notes
- Gmail connector works (read + create draft). Housing-request leads sometimes withhold contact → reply via FF platform; direct-message/booking leads carry a usable email.
- `meta.yaml: processed_message_ids` prevents re-processing leads. Last run 2026-07-18.
