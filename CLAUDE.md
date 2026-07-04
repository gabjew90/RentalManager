# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this repository is

Documentation-only repository: the operations system for a solo-run furnished mid-term rental business (two units — Fremont and Newark, CA; 2–6 month stays for traveling professionals; marketed on Furnished Finder and Airbnb). There is no application code, build, lint, or test tooling.

The system's core objective, which every document serves: **every gap between tenancies lands in 7–14 days** — gaps over 14 days are the failure mode the system prevents; gaps under 7 days are not forced either.

## Structure

- `README.md` — overview and doc index
- `docs/system-design.md` — canonical reference: tracker field spec, demand calendar (end-date tiers), end-date engineering, channel mechanics, pricing, vacancy math, KPIs
- `docs/runbook.md` — operator checklists: weekly routine, T−N triggers keyed to lease end, escalation ladder, signing checklist
- `docs/templates.md` — message templates (IDs: P1, E1, C1, D1, I1, I2, F1, F2, F3, A1, M1) referenced from the runbook and the automation workflows
- `docs/automation.md` — LLM ops-manager architecture: Gmail as event bus, scheduled agent runs, autonomy tiers (auto / approve-to-send / human-only)
- `automation/agent/ops-manager.md` — operating instructions followed by the scheduled ops-manager agent
- `automation/workflows/` — per-task workflow specs the agent executes (currently: `ff-lead-followup.md`); a workflow file governs where it and ops-manager.md overlap
- `automation/facts/units-facts.yaml` — the only property facts the agent may state to leads autonomously; anything not in it requires operator approval before replying
- `automation/state/*.yaml` — agent-maintained state (units, inquiries, tenancies, approvals, meta). **Agent-owned:** humans edit only to seed data or flip `meta.yaml: autonomy`; every mutation is committed (git history is the audit log)

## Conventions

- **The spec stays technology-neutral; `automation/` implements it.** `docs/system-design.md`, `runbook.md`, and `templates.md` must remain implementable with a spreadsheet and calendar reminders — no tool-specific instructions there. The automation layer (`docs/automation.md`, `automation/`) is the LLM-driven implementation; it computes from the spec and must never carry its own copies of business numbers.
- **Safety-critical automation rules** (never weaken without explicit operator direction): nothing binding is sent without an explicit operator approval; block-before-yes (no date confirmation while the opposite channel's block is unconfirmed); the 30-day stay floor; low parse confidence escalates to the digest rather than guessing.
- **system-design.md is canonical.** Numbers that appear in multiple docs (trigger days like T−60/T−45, the 7–14 day gap target, term-premium percentages, end-date tier windows) are defined in system-design.md; runbook and templates must stay consistent with it. When changing any such number, update all three docs in the same commit.
- **Template IDs** (E1, D1, …) are referenced from the runbook — renaming or removing one requires updating its runbook references.
- **Fixed business context** (do not "improve" these away): Furnished Finder is lead-gen/direct-lease with no fees; Airbnb is commission-based with its own reviews/ranking; channel policy is opportunistic — date fit and net revenue pick the winning booking, not channel preference; **all stays have a 30-day floor** (Fremont/Newark treat shorter stays as STR — permit + TOT); one unit has reviews, the newer one doesn't (templates cross-reference the reviewed unit); assume no programmatic access to channel calendars — sync is manual, and the tracker is the single source of truth.
- The demand-calendar date windows in system-design.md §3 are **calibrated from 160 real FF leads (Feb 2024 – Jul 2026;** see `automation/state/calibration/summary.md`): demand is dominated by the summer tech-intern cycle (search Feb–Apr, move in May, move out Aug; ~91-day median stays), not healthcare quarters. Recalibrate annually from the inquiry log; don't revert to the old assumed cycles.
- The operator has a minimal time budget: any proposed process change must fit ~10 min/day + 30 min/week + 30 min/month.
