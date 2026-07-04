# Prototype: FF Lead Follow-up — dry run

A recorded execution of `automation/workflows/ff-lead-followup.md` against fixture data. **All names, rates, and property facts here are demo data; nothing was sent anywhere.**

- `state-seed/` — demo versions of `units.yaml` and `units-facts.yaml` (what the real files look like filled in)
- `fixtures/` — four synthetic FF lead emails, one per triage branch:
  1. dates fit, Red end date → I1 + C1 engineered-date counter (+ SMS draft)
  2. dates fit, Green end date → clean I1 with review cross-reference
  3. requested unit occupied, other unit fits → A1 cross-sell — and it **competes** with lead 2 for the same unit
  4. sub-30-day request → A1 decline variant
- `run-2026-07-04/` — the agent's output: updated inquiry state, outbox drafts, and `digest.md` (the email the operator would receive)

Run mode: `supervised` (as at go-live), so every draft queues for approval — nothing is auto-sent. In `standing` mode, items 1–4 below would have gone out within 2h and only the competing-lead decision would need the operator.

To re-run: give the agent this directory's seed state + a new fixture and ask it to execute the workflow.
