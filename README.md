# RentalManager

Operations system for two furnished mid-term rental units (2–6 month stays, traveling professionals) in Fremont and Newark, CA, marketed on Furnished Finder (direct leases) and Airbnb (platform bookings). Run by a solo operator with a low time budget — everything here is checklist-driven and technology-neutral (a spreadsheet + calendar reminders is a full implementation).

**The one rule the whole system serves:** every gap between tenancies lands in the **7–14 day** range. Over 14 days is the failure mode; under 7 is not forced either.

## Documents

| Doc | What it is |
|---|---|
| [docs/system-design.md](docs/system-design.md) | The reference design: tenancy tracker spec, Bay Area demand calendar with green/red end-date windows, end-date engineering rules, channel mechanics, term-based pricing, vacancy math, KPIs. |
| [docs/runbook.md](docs/runbook.md) | What to actually do: weekly 30-minute routine, T−75 → move-in trigger checklists, escalation ladder for at-risk gaps, signing checklist. |
| [docs/templates.md](docs/templates.md) | Fill-in messages for every trigger: extension offers, departure confirmation, inquiry replies (both channels, with review cross-referencing for the newer unit), end-date counter-offers. |

## How it runs

1. **At every signing:** engineer the end date into a strong demand window (never accept an arbitrary one), apply the term-premium pricing, set calendar reminders for the T−75/−60/−45/−30/−14/−7 triggers.
2. **Weekly, 30 minutes:** update the tracker, fire due triggers, audit both channel calendars against it.
3. **Daily, under 10 minutes:** answer inquiries same-day from templates.

Time budget: ~10 min/day + 30 min/week + 30 min/month.
