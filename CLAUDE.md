# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this repository is

Documentation-only repository: the operations system for a solo-run furnished mid-term rental business (two units — Fremont and Newark, CA; 2–6 month stays for traveling professionals; marketed on Furnished Finder and Airbnb). There is no application code, build, lint, or test tooling.

The system's core objective, which every document serves: **every gap between tenancies lands in 7–14 days** — gaps over 14 days are the failure mode the system prevents; gaps under 7 days are not forced either.

## Structure

- `README.md` — overview and doc index
- `docs/system-design.md` — canonical reference: tracker field spec, demand calendar (end-date tiers), end-date engineering, channel mechanics, pricing, vacancy math, KPIs
- `docs/runbook.md` — operator checklists: weekly routine, T−N triggers keyed to lease end, escalation ladder, signing checklist
- `docs/templates.md` — message templates (IDs: P1, E1, C1, D1, I1, I2, F1, M1) referenced from the runbook

## Conventions

- **Technology-neutral:** the system must be implementable with a spreadsheet and calendar reminders. Don't introduce tool- or vendor-specific instructions into the docs; if software is ever added to this repo, it implements the spec — the spec stays neutral.
- **system-design.md is canonical.** Numbers that appear in multiple docs (trigger days like T−60/T−45, the 7–14 day gap target, term-premium percentages, end-date tier windows) are defined in system-design.md; runbook and templates must stay consistent with it. When changing any such number, update all three docs in the same commit.
- **Template IDs** (E1, D1, …) are referenced from the runbook — renaming or removing one requires updating its runbook references.
- **Fixed business context** (do not "improve" these away): Furnished Finder is lead-gen/direct-lease with no fees and suits long anchor tenancies; Airbnb is commission-based with its own reviews/ranking and suits short gap-fills; one unit has reviews, the newer one doesn't (templates cross-reference the reviewed unit); assume no programmatic access to channel calendars — sync is manual, and the tracker is the single source of truth.
- The demand-calendar date windows in system-design.md §3 are derived from stated contract cycles (healthcare Jan/Apr/Jul/Oct; academic May–Sep), marked for recalibration against the operator's inquiry log — treat them as tunable parameters, not facts.
- The operator has a minimal time budget: any proposed process change must fit ~10 min/day + 30 min/week + 30 min/month.
