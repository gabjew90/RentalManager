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

## 3. Demand calendar (SF Bay, mid-term furnished)

Demand clusters around predictable move-in waves:

| Move-in wave | Driver | When they search |
|---|---|---|
| Jan 2–15 | Healthcare Q1 contracts | Nov – late Dec |
| Apr 1–14 | Healthcare Q2 contracts | Feb – Mar |
| May 15 – Jun 30 | Interns, academics, summer relocations | Mar – May |
| Jul 1–14 | Healthcare Q3 contracts (peak season) | May – Jun |
| Aug 15 – Sep 30 | Academic year starts | Jun – Aug |
| Oct 1–14 | Healthcare Q4 contracts | Aug – Sep |

A **good end date is one that sits 7–14 days before a move-in wave.** That yields the end-date tiers used everywhere in this system:

| Tier | End-date windows | Why |
|---|---|---|
| 🟢 Green | **Mar 18 – Sep 23** | Rolling demand: Apr wave, then May–Sep continuous (interns + Jul wave + academic), then Oct 1 wave catches late-Sep ends. |
| 🟡 Yellow | **Dec 19 – Jan 5** · **Mar 1 – 17** · **Sep 24 – 30** | Adjacent to a wave but with holiday risk (Dec/Jan) or slightly early/late timing. Workable with early pre-marketing. |
| 🔴 Red | **Oct 1 – Dec 18** · **Jan 6 – Feb 28** | Post-wave dead zones. An Oct 20 end date means the next big wave is Jan — a 10-week structural gap unless bridged. Worst stretch: mid-Oct through Nov. |

Calibration: these windows are derived from the stated contract cycles, not measured data. Once the inquiry log has two or three quarters of desired-start dates, adjust the boundaries from observed demand.

## 4. End-date engineering

Never passively accept a tenant-proposed end date. At every lease signing and extension:

1. Compute the tier of the proposed end date.
2. If Green → accept.
3. If Yellow/Red → counter with a prorated term that shifts the end into Green (template C1). Prefer **lengthening** the stay (more revenue, and tenants rarely object to "we can do 3.5 months instead of 3"). Shift by whatever it takes; odd terms are fine — price the extra days at straight daily proration (monthly ÷ 30).
4. If the tenant genuinely can't flex (contract-tied dates), accept but price the tail risk: apply the next term-premium tier up (§6), and flag the row Yellow/Red so pre-marketing starts at T−60 instead of T−45.

Red-window rule of thumb: a tenancy ending Oct–Feb should either be extended through the winter (ideal: end mid-March or later) or end **Dec 19 – Jan 5** to catch the January healthcare wave. An end date in mid-October or November is the single most expensive date mistake this business can make.

## 5. Channel mechanics and coordination

Two channels with fixed, different mechanics:

| | Furnished Finder | Airbnb |
|---|---|---|
| Model | Lead gen → direct lease, no booking fee | Booking platform, commission, own reviews/ranking |
| Best for | **Anchor tenancies** (3–6+ months) | **Gap fills and bridges** (2 weeks – 2 months), and anchors when a strong booking appears |
| Payments | Direct (deposit + rent) | Platform-handled |
| Reviews | Portfolio-level trust (cross-reference works) | Per-listing ranking; response rate/time affect visibility |

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
| < 2 months (bridges) | +20% | Airbnb gap fills |

Adjustments, applied in order:

1. **Term premium** (table above).
2. **Channel gross-up:** Airbnb advertised price = target net ÷ (1 − Airbnb commission %), so both channels net the same. Direct FF leases carry no gross-up — that margin is why FF anchors are preferred.
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
