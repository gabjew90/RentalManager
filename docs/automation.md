# Automation Architecture — LLM Ops Manager

Automated implementation of [system-design.md](system-design.md). The spec stays technology-neutral; this document describes the software that executes it. Operator involvement is reduced to **approving drafted actions** plus the few physical clicks that live behind channel logins.

## The constraint that shapes everything

Neither channel exposes an API to a two-unit operator: Furnished Finder has none; Airbnb has none for individual hosts (scraping violates ToS and risks the account — off the table). But **both channels push every event into the operator's Gmail**: FF lead emails (with the lead's name, dates, phone, email), Airbnb inquiry/booking/message notifications, and tenant email threads.

Therefore: **Gmail is the event bus. The LLM agent is the consumer. This git repository is the database. The operator is an approval step, not a worker.**

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
| **Human-only (physical)** | Pasting Airbnb replies into the app (agent drafts; no API to send), blocking dates on the Airbnb calendar / updating the FF availability date (agent gives the exact instruction: "Block Oct 3 – Dec 1 on Airbnb ✅?"), signing leases, deposits, turnover work | Agent generates the exact action + tracks that you confirmed it done — the block-before-yes rule is enforced by the agent refusing to confirm dates to anyone until you've confirmed the other channel is blocked |

Expected operator load: **~3–5 one-tap approvals per week, a couple of Airbnb pastes, and channel-calendar clicks a few times a year.** No spreadsheet, no manual tracking, no remembering trigger dates — the agent computes and fires them.

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
