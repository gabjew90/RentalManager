# Workflow: Furnished Finder Lead Follow-up

Owns a lead from FF email arrival until it exits to **application/background check** (next workflow), **hold/signing**, or **dead/dormant**. Executed by the ops-manager agent every run; referenced from `automation/agent/ops-manager.md`.

Why this workflow is aggressive on speed: an FF lead is a tenant who messaged several landlords at once. The first substantive, personalized response usually wins the conversation. Target: **first touch within 2 hours, 8am–9pm PT.**

## ⭐ MVP scope (current build)

Three steps only, all output as **drafts for operator approval** — nothing auto-sends:

1. **Intake** — parse FF lead emails from Gmail into `inquiries.yaml` (§1).
2. **Evaluate** — the end-date-engineering core (§2a below): availability fit, end-date tier, gap projection, and a dates-only counter-proposal when the requested window lands badly.
3. **Draft initial outreach** — one personalized first-touch draft per lead (+ SMS paste when a phone number exists).

**Pricing rule for MVP: the listed price is the only price.** No term premiums, no computed quotes, no discounts at the lead stage — drafts reference the listing's advertised monthly rate as-is (proration to exact dates at that same listed rate is fine to mention). The term-ladder pricing in system-design §6 applies at lease-negotiation time, not in outreach, until pricing is brought into scope.

Deferred to later phases: the no-reply follow-up cadence (§3), the facts-bounded qualification loop (§4), auto-send/standing autonomy, background check, lease drafting.

### §2a Evaluate — lead scorecard (the "calendar management" step)

For every parsed lead, compute and store:

| Check | From | Output |
|---|---|---|
| Availability fit | requested window vs `units.yaml` tenancies + turnover buffers | fits / conflicts / fits-other-unit |
| Start alignment | requested move-in vs current tenancy end | projected gap days + 🟢🟡🔴 vs the 7–14 target |
| **End-date tier** | requested move-out vs demand windows (system-design §3) | GREEN / YELLOW / RED |
| **Slot fit** | requested window vs the three-tenancy template (system-design §3): summer May 15–Aug 16 · fall Sep 1–Dec 31 · winter Jan 5–May 8 | which slot it fills, and whether its dates need a counter to fit the slot boundaries (esp. winter-cohort ends → Apr 28–May 8, April bridges → end ≤ May 8) |
| Date counter | if end is YELLOW/RED or start misaligned | engineered alternative window (dates only, listed rate, prorated) — this goes *into the outreach draft* as an option, never as a condition |
| Quality signals | occupants, pets vs policy, employer/facility, stay length, message tone | free-text note |
| Recommendation | all of the above | `reply` / `reply_with_date_counter` / `offer_other_unit` / `decline_sub30` / `escalate_to_operator` |

The scorecard is the evaluation deliverable: it appears with the draft in the digest so the operator sees *why* the draft says what it says.

## 1. Intake and parsing

Trigger: Gmail from `furnishedfinder.com` — lead notification or listing message. Extract:

| Field | Notes |
|---|---|
| name, email, phone | FF leads include direct contact info — this is FF's model |
| unit | Which listing (or a general housing request matching our area) |
| desired start / end | End may be implied by "13-week contract" etc. — compute it |
| party, pets, budget, free text | When present |

Parse confidence < high on dates or contact → digest item with raw email, no auto-send. Every lead gets a row in `inquiries.yaml` (id, all fields, `end_tier`, `status: new`, `touches: []`).

## 2. Triage branches (first touch; in MVP all are drafts — Auto tier only in a later phase)

Evaluate against `units.yaml` availability and the demand tiers:

**Lead vs inquiry — pick the register first.** A **housing-request lead** is the traveler casting a wide net (broadcast to many landlords; may not have seen this listing) → first touch is **L1 (light): friendly hello + "check out the listing" + one honest commute line if relevant.** No qualification/screening/pricing in the first message. A **booking inquiry / direct message** is targeted at this property (high intent) → qualify with I1/I2. The cases below apply the *matching* register (L1 for leads, I1 for inquiries).

| Case | Action | Template |
|---|---|---|
| Dates fit, unit available, end date Green | Qualification reply, reference the listed monthly rate | I1 |
| Dates fit, end date Yellow/Red | I1 **with the engineered end-date counter folded in** (prorated alternative that lands Green) | I1 + C1 |
| Requested unit unavailable, other unit fits | Offer the alternative unit (cross-reference reviews if offering Newark) | A1 |
| Neither unit fits their window | Polite no + ask to keep on file for their next contract; log `dormant` with their timing | A1 (variant) |
| Stay request < 30 days | Decline (STR floor — hard rule), suggest 30-day minimum alternative | A1 (variant) |

First touch goes to **email always**; if the lead included a phone number, also produce a ready-to-send **SMS version (≤300 chars)** as a one-tap paste for the operator — texts get dramatically faster replies from travel nurses. (If an SMS rail like Twilio is added later, SMS joins the Auto tier; until then it's an optional operator assist, never a blocker.)

Rate references: **listed price only** (see MVP scope) — the advertised monthly rate on the listing, optionally prorated to exact dates at that same rate. No term-ladder math in outreach.

## 3. Follow-up cadence (no reply) — DEFERRED (later phase)

Max **3 unsolicited touches total**, then dormant. Stop instantly on any reply or opt-out.

| Touch | When | Template | Content angle |
|---|---|---|---|
| 1 | ≤ 2h | I1 (+SMS paste) | Answer + qualify + rate |
| 2 | +48h | F2 | Short bump: still looking? happy to hold dates 48h once confirmed |
| 3 | +6 days | F3 | Value add: reviews link (Fremont history), date flexibility / exact-contract proration, one photo highlight |
| — | dormant | | Resurfaces only via T−30 pipeline re-contact (F1) or if their stated timing matches a future opening |

`inquiries.yaml` per lead: `last_touch`, `next_followup_due`, `touches[]` — the agent computes due follow-ups every run; nothing relies on memory.

## 4. Reply → qualification loop — DEFERRED (later phase)

On any reply, `status: in_conversation`. The agent converses over the email thread within strict bounds:

- **May answer autonomously:** anything in `automation/facts/units-facts.yaml` (the approved-facts file: parking, pets, utilities, furnishings, workspace, laundry, deposit, standard lease terms) plus list-price quotes and availability dates. **Facts file silent on it → Approve-to-send.** Never improvise a fact about the property.
- **Qualification checklist to complete:** exact dates (start *and* end) · occupants + pets · contract/assignment confirmed (agency + facility for travel nurses; offer letter for relocations) · rate acknowledged · special needs (parking, desk, EV, etc.).
- Checklist complete → `status: qualified` → **digest/Telegram item for the operator**: lead summary, date-fit score (rent minus expected gap cost of their end date, per design §6), recommendation, and proposed next step — a scheduled call/video tour (offer slots from `units-facts.yaml: tour_availability`) or straight to application if the lead asks.

## 5. Exits

| Exit | Handling |
|---|---|
| → **Application / background check** | Operator approves; hand off to the background-check workflow (`status: application_sent`). 48h clock starts only at **hold** (deposit-pending), never at application |
| → **Hold** | Approve-to-send; 48h max; agent auto-queues release-or-sign at expiry; block-before-yes applies |
| → **Dead** | Log reason (`price`, `dates`, `ghosted`, `chose_elsewhere`, `unqualified`) — this feeds the demand-window recalibration and pricing review |
| → **Dormant** | Timing mismatch; keep contact + timing for future openings |

**Competing leads for overlapping dates:** never first-come-first-served by default — rank by date-fit score and pipeline risk, present the ranking in the digest, operator picks. The agent keeps runner-ups warm ("expecting an answer on the dates shortly") without promising anything.

## 6. Metrics (weekly digest)

First-touch time · touch→reply rate (by touch #) · reply→qualified · qualified→application · dead reasons. These tell us whether the cadence timing and the F2/F3 content earn their keep.
