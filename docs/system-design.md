# Operations System Design

Technology-neutral design for running two furnished mid-term rental units (Fremont and Newark, SF Bay Area) as a low-touch side business. The system's single job: **make every gap between tenancies land in the 7–14 day range.** Gaps over 14 days are the failure mode; gaps under 7 days are avoided too (turnover comfort, no need to force them).

Everything else in this document — date engineering, extension triggers, pre-marketing, pricing — exists to serve that one constraint.

## 1. Why gaps happen (and what this system attacks)

Revenue loss comes from vacancy between tenancies, driven by date misalignment, not turnover work (turnover fits easily in 1–2 weeks):

1. **End dates land in weak demand windows** → the next-tenant search takes 4–10 weeks instead of 1–2.
2. **Extensions aren't proactively offered** → tenants who would have stayed leave.
3. **Marketing starts after move-out** instead of 45–60 days before.

The system fixes each with a mechanical rule: engineer end dates at lease signing (§4), trigger extension offers at day −60 (runbook), and start pre-marketing at day −45 (runbook).

## 2. Tenancy tracker (single source of truth)

One tracker — a spreadsheet is sufficient — with three tabs. The tracker, not either listing channel, is the authoritative calendar. Every date change anywhere gets entered here first.

### Tab 1: Units (dashboard — one row per unit)

| Field | Definition |
|---|---|
| Unit | Fremont / Newark |
| Current tenant | Name |
| Channel | Furnished Finder (direct lease) / Airbnb |
| Lease start | Date |
| Lease end | Date (update on every signed extension) |
| End-date tier | Green / Yellow / Red per §3 — computed from lease end |
| Extension status | `not_yet_due` → `offered` → `accepted` / `declined` / `month_to_month` |
| Departure confirmed | Yes/No — written confirmation received (template D1) |
| Next tenant / move-in | Name + confirmed move-in date, or blank |
| **Gap days** | Next move-in − lease end. Blank next move-in inside T−30 counts as Red. |
| Gap status | 🟢 7–14 · 🟡 0–6 (too tight) or 15–21 (act now) · 🔴 >21 or unbooked inside T−30 |
| Next trigger + date | The next runbook trigger due (e.g., "T−60 offer → Aug 4") |

### Tab 2: Tenancy log (one row per completed or signed tenancy)

Unit, tenant, channel, start, end, monthly rate, term-premium tier applied, actual gap days that followed, notes. This is the KPI source.

### Tab 3: Inquiry log (lightweight — one line per inquiry)

Date, channel, name, desired start/end, end-date tier of *their proposed end date*, status (replied / qualified / toured / signed / dead). Filled in as part of answering the inquiry, not as separate admin.

## 3. Demand calendar (SF Bay, mid-term furnished) — CALIBRATED FROM LEAD DATA

**Source: 160 real FF leads, Feb 2024 – Jul 2026** (`automation/state/calibration/summary.md`). The originally assumed healthcare-quarterly model was not supported; these units live on the **tech/intern cycle**:

| Wave | Who | When they search | Move in | Move out |
|---|---|---|---|---|
| **Summer interns/new grads (dominant — ~40% of all dated demand)** | Meta, Google, Tesla, Intel, Stanford; often 2-person pairs | **Feb–Apr** (51% of annual lead volume; March alone is the peak) | **May** (44 of 107 dated leads) | **Mid/late Aug** (45 of 102) |
| Fall movers (secondary) | Tech/corporate relocations, some academic | Oct bump | Sep–Oct, some Jan | Mar–Apr (a real Sep→spring long-stay cohort) |
| Healthcare travelers | ~4% of leads | scattered | scattered | scattered |

Median requested stay: **~91 days**. Search volume **May–July is near zero** (3/6/1 leads per calendar month across 2.5 years).

**End-date tiers** (a good end date opens the unit into a period when people are actually searching to move in soon):

| Tier | End-date windows | Why |
|---|---|---|
| 🟢 Green | **Apr 1 – May 15** | Unit opens straight into the May wave — the one high-certainty fill of the year |
| 🟡 Yellow | **Aug 15 – Oct 10** · **Dec 15 – Jan 20** · **Feb 15 – Mar 31** | Aug/Sep: real Sep–Oct demand but everyone's intern-vacated units hit the market at once · Dec/Jan: modest Jan-mover cohort · late winter: peak *search* season but most searchers want May — gap risk until the wave |
| 🔴 Red | **May 16 – Aug 14** (worst: June–July) · **Oct 11 – Dec 14** · **Jan 21 – Feb 14** | A June/July opening faces an empty search market — the single most expensive end-date mistake for these units |

**Canonical unit year** that end-date engineering steers toward: **May → mid-Aug intern stay**, then **Sep → early/mid-April anchor** (one long stay, or Sep–Dec + Jan–Apr), ending in the Green window to catch the next May wave.

Calibration notes: lead-arrival volume is partly confounded by when the listings were open; the tiers lean on desired move-in/move-out months, which aren't. Recalibrate annually from the inquiry log (n grows with every lead the agent processes).

## 4. End-date engineering

Never passively accept a tenant-proposed end date. At every lease signing and extension:

1. Compute the tier of the proposed end date.
2. If Green → accept.
3. If Yellow/Red → counter with a prorated term that shifts the end into Green (template C1). Prefer **lengthening** the stay (more revenue, and tenants rarely object to "we can do 3.5 months instead of 3"). Shift by whatever it takes; odd terms are fine — price the extra days at straight daily proration (monthly ÷ 30).
4. If the tenant genuinely can't flex (contract-tied dates), accept but price the tail risk: apply the next term-premium tier up (§6), and flag the row Yellow/Red so pre-marketing starts at T−60 instead of T−45.

Red-window rules of thumb (from the calibrated calendar in §3):
- **Never let a tenancy end in June or July** — the search market is empty until fall. An intern stay should run through at least mid-August; extensions that push a May/June end into Aug–Sep are worth real concessions.
- A tenancy ending Oct–Feb should either be extended to land **Dec 15 – Jan 20** (Jan-mover cohort) or, better, carried all the way to **early April** (Green) — that's the canonical Sep→April anchor.
- The most valuable single end window of the year is **Apr 1 – May 15**; when in doubt, engineer toward it.

## 5. Channel mechanics and coordination

Two channels with fixed, different mechanics:

| | Furnished Finder | Airbnb |
|---|---|---|
| Model | Lead gen → direct lease, no booking fee | Booking platform, commission, own reviews/ranking |
| Strengths | No-fee direct leases; strong for longer tenancies | Fast fills; strong for shorter stays and quick turnarounds |
| Payments | Direct (deposit + rent) | Platform-handled |
| Reviews | Portfolio-level trust (cross-reference works) | Per-listing ranking; response rate/time affect visibility |

**Channel policy is opportunistic, not hierarchical.** Both channels open simultaneously at T−45 on equal footing; the winning booking is whichever nets more after the date-fit override (§6) — channel preference never decides. The Airbnb gross-up (§6) makes net revenue comparable across channels, and term premiums (not min-stay gating) steer the mix toward longer stays.

**30-day floor on all stays, both channels.** Fremont and Newark treat sub-30-day stays as short-term rentals (permit + transient occupancy tax). This system never books under 30 days; the shortest instrument is a 1-month bridge. Airbnb min-stay is set to 30 days permanently.

**Anti-double-booking protocol** (calendars can't sync programmatically — assume manual):

- The tracker is authoritative; both channel calendars are projections of it.
- **Block-before-yes:** the moment any booking or lease is verbally agreed on one channel, block those dates on the *other* channel — before sending the confirmation message.
- Furnished Finder leads get a **48-hour soft hold** maximum; only a signed lease + deposit converts a hold to a block. Never hold Airbnb calendar dates for an unsigned FF lead beyond 48h.
- Weekly routine (runbook) includes a both-channels calendar audit against the tracker.

**Review asymmetry:** the established unit's review history is a portfolio asset. Every inquiry reply for the unreviewed unit cross-references it ("same owner and standard as our [other city] unit — see its reviews"). On Airbnb, protect the ranking inputs: respond to every inquiry within a few hours (templates make this a 2-minute task) and never let a request expire.

## 6. Pricing

Base rate = the 6+ month monthly rate per unit, reviewed monthly against comps (runbook). Everything else is computed from it:

| Term | Price vs base | Rationale |
|---|---|---|
| 6+ months | Base | Anchor tenancy, minimal turnover load |
| 4–5 months | +5% | |
| 2–3 months | +12% (band 10–15%) | Turnover-heavy bookings fund the vacancy risk they create |
| 1–2 months (bridges) | +20% | Shortest bookable term — the 30-day floor (§5) means nothing shorter exists |

Adjustments, applied in order:

1. **Term premium** (table above).
2. **Channel gross-up:** Airbnb advertised price = target net ÷ (1 − Airbnb commission %), so both channels net the same and can be compared purely on dates and net revenue (the opportunistic policy in §5 depends on this).
3. **Season:** stays mostly covering Nov–Feb may take −5% (or bundled utilities) to close; May–Sep peak can carry +5%.
4. **Date-fit override:** an inquiry whose end date is Green is worth more than its rate suggests; one ending Red is worth less. When comparing two inquiries, subtract the expected gap cost each one's end date creates (§7). A slightly cheaper tenant ending March 20 beats a pricier one ending October 25.

## 7. Vacancy math (drives extension pricing)

- Daily revenue ≈ monthly rate ÷ 30 (≈ 3.3% of a month per day).
- The intended gap is 7–14 days (call it 10). **Every expected gap day beyond 10 is pure loss.**
- **Extension concession cap:** if declining an extension leads to an expected gap of G days, any concession costing less than `(G − 10) × daily rate` is profitable. Example: a tenant ending Nov 1 who declines to extend probably creates a 45+ day gap → up to ~35 days of rent (~115% of one month) can rationally be spent across the extension term to keep them. In practice a 3–5% discount on a 2–3 month extension costs a fraction of that.
- Standard extension pricing: extend at **current rate** (no increase) as the default offer; add a small discount (3–5%) only when the current end date is Red and the extension moves it to Green. Month-to-month holdover is allowed at **+8–10%** with 30 days' written notice — the premium prices the date uncertainty it creates.

## 8. KPIs (reviewed monthly, 5 minutes, from the tenancy log)

| KPI | Target |
|---|---|
| Gap days per turnover | 7–14, never >14 |
| Trailing-12-month occupancy per unit | ≥ 75% breakeven (~9 months); above is premium |
| Extension offer acceptance | Track; expect ≥ 30–40% |
| % of signed end dates in Green tier | ≥ 80% |
| Inquiry first-response time | Same day, ideally < 3 waking hours |

## 9. Operating time budget

- **Daily:** inquiry replies only, from templates — under 10 minutes.
- **Weekly:** one 30-minute routine (runbook §1): update tracker, fire due triggers, audit both calendars.
- **Monthly:** 30 minutes — comp/pricing check, KPI glance, demand-calendar calibration.

Nothing in this system requires software beyond a spreadsheet, two channel dashboards, and calendar reminders for triggers. If a tool with reminders/automation is adopted later, it implements this spec; the spec doesn't change.
