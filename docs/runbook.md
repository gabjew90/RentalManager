# Operator Runbook

Checklist-driven execution of the [system design](system-design.md). Two rhythms: a **weekly 30-minute routine**, and **date-triggered checklists** keyed to each unit's lease end date (T = current lease end). Set a calendar reminder for each trigger the day a lease or extension is signed — that one habit runs the whole system.

Message templates referenced here (E1, D1, C1, …) live in [templates.md](templates.md).

## 1. Weekly routine (30 min, same day each week)

1. **Update the tracker** — new inquiries, date changes, signatures, deposits.
2. **Recompute per unit:** end-date tier, gap days, gap status.
3. **Fire any due triggers** (section 2) — the tracker's "next trigger" column tells you what's due.
4. **Calendar audit:** open both Furnished Finder and Airbnb calendars, compare against the tracker, fix any drift. (Anti-double-booking backstop.)
5. **Pipeline check:** any unit inside T−45 without a confirmed next move-in → escalation ladder (section 3).

Monthly (add 30 min once a month): comp check on 3–5 similar listings per unit → adjust base rate if drifted; glance at KPIs; adjust demand-calendar windows if the inquiry log disagrees with them.

## 2. Trigger checklists (per unit, T = lease end)

### T−75 · Plans check-in
- [ ] Friendly check-in with tenant (template P1): how's the stay, any sense of plans after the lease?
- [ ] Confirm the tracker's end-date tier for T. If T is Red, treat the T−60 extension offer as high-priority and pre-draft it now.

### T−60 · Extension offer
- [ ] Send extension offer (template E1): default = extend at current rate; add 3–5% discount only if it moves a Red end date to Green (see design §7 for the concession cap).
- [ ] Engineer the extension end date — offer terms that land in a Green window, prorated to odd lengths if needed.
- [ ] State a decision-by date of **T−45** in the offer.
- [ ] Log `Extension status = offered`.

### T−45 · Departure confirmation + pre-marketing launch
Runs only if extension declined or no answer by deadline. If T is Red-tier, run this whole block at T−60 instead. **Seasonal override:** if the opening targets the May intern wave (end date in the Green window), have listings live with correct availability dates by **early February** regardless of T−45 — May movers search Feb–Apr, 1–3 months ahead.
- [ ] Written departure confirmation (template D1) — locks the date and starts the countdown cleanly.
- [ ] Tracker: `Departure confirmed = Yes`.
- [ ] **Both channels, same day, equal footing** (channel policy is opportunistic — design §5):
  - **Furnished Finder:** set availability date = T+7. Refresh listing (rate per pricing table, photos, description).
  - **Airbnb:** open calendar from T+7 onward; keep T…T+6 blocked for turnover. Min-stay stays at the permanent 30-day floor — term premiums, not min-stay gating, steer toward longer stays.
- [ ] Set pricing for the projected next-tenancy season (design §6).
- [ ] Start working inquiries same-day using templates I1/I2.

### T−30 · Pipeline review (only if next move-in not confirmed)
- [ ] Rate check against comps; correct if above market.
- [ ] Verify Airbnb is at the 30-day min-stay floor (never lower — local STR rules) and priced per the season.
- [ ] Furnished Finder: refresh/bump the listing; re-contact every qualified-but-stalled inquiry from the log (template F1).

### T−14 · Bridge mode (only if still unbooked)
- [ ] Actively pitch **1-month bridge stays** at the +20% tier (both channels; Airbnb usually fills these fastest). Never below 30 days.
- [ ] **Engineer the bridge end date:** size the bridge (1, 1.5, 2 months — prorate freely) so *its* end lands 7–14 days before the next demand wave — a bridge that ends Nov 20 just recreates the problem.
- [ ] Consider −5% on the listed price for longer terms; note it in the log.

### T−7 · Turnover prep
- [ ] Confirm cleaner/maintenance/staging for T+1 … T+5.
- [ ] Send tenant move-out logistics (template M1): time, key return, deposit process.

### T (move-out) → T+7 · Turnover
- [ ] Move-out inspection; start deposit clock per lease/state law.
- [ ] Deep clean, maintenance punch list, staging, photo refresh if the unit changed.
- [ ] Target ready-date: T+5, leaving buffer inside even a 7-day gap.

### Move-in −3 · Arrival prep
- [ ] Confirm arrival time and access instructions with incoming tenant.
- [ ] Walkthrough: utilities on, Wi-Fi live, supplies stocked.
- [ ] Both channel calendars reflect the new tenancy (block-before-yes should have already done this — verify).

## 3. Escalation ladder (unit inside T−45, no confirmed next move-in)

Work down the ladder; each step is roughly a week apart and already embedded in the T−30/T−14 triggers above:

1. **Price** — comp check, correct the rate.
2. **Widen** — FF bump + re-contact stalled leads; confirm Airbnb is at the 30-day floor and seasonally priced.
3. **Bridge** — 1–2 month stay at +20%, end-date engineered to the next wave (30-day floor always holds).
4. **Buy the date** — for a strong applicant with slightly-off dates, discount or prorate to close; per design §7, anything cheaper than the gap it prevents is profitable.

Never hold out past T−14 for a full-price anchor while refusing bridges — a structurally empty unit waiting for January is the exact failure this system exists to prevent.

## 4. Signing checklist (any new lease or extension, either channel)

- [ ] Proposed end date tiered; countered via C1 if Yellow/Red (design §4).
- [ ] Term premium applied (design §6); Airbnb gross-up if applicable.
- [ ] FF direct lease: signed lease + deposit received before dates are promised. 48h max soft hold.
- [ ] **Block-before-yes:** other channel's calendar blocked before the confirmation message is sent.
- [ ] Tracker updated: new row in tenancy log, Units dashboard refreshed.
- [ ] Calendar reminders created for T−75 / T−60 / T−45 / T−30 / T−14 / T−7.
