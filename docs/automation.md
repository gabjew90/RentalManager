# Automation Architecture — LLM Ops Manager

Automated implementation of [system-design.md](system-design.md). The spec stays technology-neutral; this document describes the software that executes it. Operator involvement is reduced to **approving drafted actions** plus the few physical clicks that live behind channel logins.

## The constraint that shapes everything

Neither channel hands an API to a two-unit operator directly — but the gap is bridgeable at several rungs, and the architecture is staged accordingly:

| Rung | Mechanism | Cost / risk | What it unlocks |
|---|---|---|---|
| 0 | **Gmail ingest** — both channels push every event (FF leads with full contact info, Airbnb notifications) into email | Free, zero risk | Baseline event bus; FF leads genuinely arrive as email, so email *is* FF's inquiry API |
| 1 | **Airbnb iCal two-way sync** — Airbnb exports each listing calendar as an iCal URL and imports external `.ics` feeds, auto-blocking their dates. The agent publishes a feed per unit (`automation/calendar/<unit>.ics`, statically hosted); Airbnb polls it every few hours | Free, ToS-clean | Automated block-before-yes on Airbnb + booking reads. No clicks, no browser |
| 2 | **Official Airbnb API partner as bridge** — individual hosts get no API, but approved channel managers (Hospitable, Hostaway, Lodgify) do, and expose their own APIs/webhooks. Connect Airbnb → bridge once; agent consumes webhooks and **sends Airbnb messages programmatically, legitimately** | ~$30–40/mo, ToS-clean | Real-time events instead of polling; deletes the paste-into-app human step; calendar + pricing writes |
| 3 | **Browser control** (Playwright / computer use) — for surfaces nothing else reaches. Appropriate for Furnished Finder's availability-date field (rare, simple, low bot-detection stakes). Last resort for Airbnb: the account is the channel *plus* the review asset, and rung 2 makes it unnecessary | Free; account risk scales with target's enforcement | FF listing updates without the operator |

**Operator interface:** a Telegram bot with inline approve/deny buttons (instant push, ask-anytime commands) is the target; the email digest is the fallback and works from day one with zero extra infrastructure. A self-hosted agent gateway (e.g., OpenClaw: Telegram + cron + browser + Gmail in one runtime) is a valid alternative host for the whole manager — trade-off: you own its security posture, and browser credentials + untrusted inbound tenant messages make prompt injection a live threat there. The approval gate on binding actions is the defense and is non-negotiable in every variant.

**Invariants across all rungs:** the LLM agent is the manager, this git repository is the database, the operator is an approval step — the rungs only change how events arrive and how actions go out.

Rollout: **rung 0 + email digest ships first** (it needs only Gmail auth), rung 1 the first week, rung 2 + Telegram as the standing state. Rung 3 only for FF.

## Components

| Component | Implementation |
|---|---|
| Event source | Gmail (read via connector): FF leads, Airbnb notifications, tenant replies, operator approval replies |
| Manager | Claude agent runs on a schedule (every 2 hours, fresh session per run), following [automation/agent/ops-manager.md](../automation/agent/ops-manager.md) |
| State / database | YAML files in `automation/state/` — units, inquiries, tenancies, pending approvals. Every change is a git commit → full audit history for free |
| Business rules | `docs/system-design.md` (tiers, pricing, vacancy math) — the agent computes from the spec, never invents numbers |
| Drafts | `docs/templates.md` — every outbound message starts from a template ID |
| Operator interface | One digest email per run (only when something needs attention) + push for urgent items. Operator replies in plain English: "1 yes, 2 no, 3 edit: …" |
| Scheduler | Claude Code Remote triggers: ingest run every 2h; the Sunday run adds the weekly audit + KPI report |

## The run loop (every 2 hours)

1. **Ingest:** search Gmail since the last run timestamp (`state/meta.yaml`) for channel emails and tenant/operator replies.
2. **Parse → state:** new inquiries, date changes, confirmations become YAML updates, committed with structured messages (`inquiry: FF/jane-d fremont 2026-09-01..2026-12-15 tier=RED`).
3. **Process approvals:** operator replies to earlier digests are executed first (send the approved email, record the decision).
4. **Compute:** per unit — end-date tier, gap days, triggers due (T−75/−60/−45/−30/−14/−7). Any trigger due → draft its action from the matching template.
5. **Act within the autonomy policy** (below): send what's pre-approved, queue the rest.
6. **Digest:** if anything is pending or changed materially, send ONE email with numbered items — context, recommendation, full draft, and what happens on "yes". No news → no email.

## Autonomy policy (what needs your tap and what doesn't)

| Tier | Actions | Behavior |
|---|---|---|
| **Auto (standing approval)** | First-touch qualification replies to FF leads (template I1, sent from Gmail); logging; state updates; reminders; KPI reports | Executed and reported in digest — no approval wait, protects same-day response SLA |
| **Approve-to-send** | Extension offers (E1), end-date counters (C1), departure confirmations (D1), stalled-lead re-contact (F1), price changes, 48h holds, anything that commits money or dates | Drafted in full; sent only on your "yes"; every digest item has a stated default if you ignore it (usually "hold") |
| **Human-only (physical)** | Signing leases, deposits, turnover work — plus, **at rung 0 only**: pasting Airbnb replies (automated at rung 2) and calendar blocks / FF availability updates (automated at rungs 1 and 3) | Agent generates the exact action + tracks that you confirmed it done — block-before-yes is enforced by the agent refusing to confirm dates to anyone until the opposite channel's block is confirmed (by you at rung 0; by iCal/bridge state at rungs 1–2) |

Expected operator load at the standing state (rungs 1–2 live): **~3–5 approval taps per week and nothing else** — no pastes, no calendar clicks, no spreadsheet, no remembering trigger dates. At rung 0 add a couple of Airbnb pastes and the rare calendar click.

## Safety rails

- **Nothing binding goes out without explicit approval.** Auto tier is strictly non-committal first-touch replies from a fixed template.
- **Block-before-yes enforced procedurally:** the agent will not draft a date confirmation to party B while a hold for party A is open on the other channel, and won't confirm any booking until the operator confirms the opposite calendar is blocked.
- **Git as audit log:** every state mutation is a commit; a wrong parse is a `git revert`.
- **Idempotent runs:** every processed email's message-ID is recorded in state; re-runs never double-process or double-send.
- **Degradation mode:** if Gmail is unreachable or parsing confidence is low, the agent queues the raw email into the digest instead of guessing.
- **Escalation:** unit unbooked inside T−30, a gap projecting >14 days, or a possible double-booking → digest marked URGENT + push notification.

## Go-live checklist (one-time)

1. Authorize the Gmail connector for Claude (claude.ai → Settings → Connectors). Until then the agent cannot read or send email.
2. Confirm the seed data in `automation/state/units.yaml` (current tenants, lease dates, rates — placeholders until filled).
3. Create the schedule: an every-2-hours ingest trigger whose prompt points at `automation/agent/ops-manager.md`, with the Sunday run flagged for weekly audit + KPIs.
4. First supervised week: everything (including Auto tier) routes through the digest; standing approvals switch on after the operator confirms parse quality.
