# Demand calibration — 160 FF leads, Feb 2024 → Jul 2026

Method: every FF lead email (housing requests 74, direct messages 53, booking inquiries 33)
parsed for received date, desired move-in/move-out, stay length, party, employer, city.
Raw rows in batch-*.yaml. Zero parse errors.

## Headline distributions

| Metric | Result |
|---|---|
| Desired move-in month | **May: 44** · everything else 1–9 per month |
| Desired move-out month | **Aug: 45** · Mar: 12 · Apr: 10 · Sep: 10 · rest ≤ 8 |
| Lead arrival | **Feb–Apr = 51%** of all volume (Mar alone = 40) · Oct secondary bump (17) · **May–Jul ≈ dead (3/6/1)** |
| Stay length | median **91 days**; mode bucket 90–119d (45); 30–59d = 22; <30d = 1 |
| Party size | 2+ occupants = 58% of known (2: 66, 3: 14, 4: 12) |
| Employers | Tech dominates (Meta 10, Google 8, Tesla 5, Intel 3, Apple, Adobe, Snowflake…); healthcare ≈ 4% (Washington Hospital, UCSF, Lucile Packard) |
| Cities | Fremont 22, Sunnyvale 16, Mountain View 11, Palo Alto 10, Menlo Park 7 — the Dumbarton/tech corridor |

## The real demand structure

**One dominant annual wave: summer interns/new grads** — search Feb–Apr, move in May,
move out mid/late Aug, ~3-month stays, often pairs. Secondary fall activity: an October
search bump with Sep/Oct and some Jan move-ins, plus a visible Sep→Mar/Apr long-stay
cohort (move-outs Mar 12 + Apr 10). The healthcare-quarterly model assumed in the original
design is NOT supported for these units.

## Day-level arrival distributions (used to embed the 7–14 day gap in the tiers)

- May move-in days: spread 1–30, **median May 16**, big cluster May 16–17, secondary clusters May 1 and May 30 (n=44)
- Aug move-out days: spread 1–31, **median Aug 16** (n=45)
- Sep move-ins: Sep 1 ×2, then mid/late Sep (n=9) · Oct move-ins: Oct 1 + mid/late Oct (n=9) · Jan move-ins: spread Jan 1–23 (n=9) · Apr move-ins sparse (n=9)

## Empirical end-date tiers (supersede the assumed ones)

Tiers are constructed so **end date + 7–14 days (the turnover gap) lands on observed arrivals**:

| Tier | End-date windows | Why |
|---|---|---|
| 🟢 GREEN | **Apr 17 – May 23** (sweet spot **Apr 28 – May 8**: ≥1 week of turnover before the dominant mid-May arrivals) | End + 7–14d hits the May arrival wave directly. Early April ends excluded (Apr arrivals barely exist → ~4 weeks idle); ends after ~May 9 only catch the late-May tail |
| 🟡 YELLOW | **Aug 15 – Oct 10** · **Dec 15 – Jan 20** · **Feb 15 – Apr 16** | Sep/Oct arrivals real but thin + intern-vacated supply flood · Jan-mover cohort (Jan 1–23) · sparse spring arrivals, May wave beyond the gap target as backstop |
| 🔴 RED | **May 24 – Aug 14** (worst: June–July) · **Oct 11 – Dec 14** · **Jan 21 – Feb 14** | Arrival deserts; June/July openings face a near-zero search market (the old assumed calendar called this window green!) |

## Canonical unit year (what end-date engineering should steer toward)

**May → mid-Aug intern stay** (premium demand, take 2-person pairs), then
**Sep → mid-April anchor** (one long stay, or Sep–Dec + Jan–Apr), ending **early-to-mid
April** to catch the next May wave. Extensions that push an end date out of June/July
into Aug-Sep, or out of Oct-Nov into Dec/Jan or all the way to April, are worth real money.

## Caveats

- Lead arrival volume partly reflects when the listings were open (availability confounder);
  desired move-in/move-out months are much less confounded and drive the tiers above.
- n=160 over ~29 months across two listings; recalibrate annually as the inquiry log grows.
- Median stay 91 days validates the 30-day floor as non-binding for most demand (<30d asks: 1 of 160).
