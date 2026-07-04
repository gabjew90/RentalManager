# Real-data parse test (2026-07-04) — PASSED

Gmail connector live. `from:furnishedfinder.com` → **201 threads**. All three FF lead
types confirmed present and parseable: housing-request leads, direct traveler messages,
booking inquiries. Contact details masked here (repo hygiene); production state files
hold full values.

## Parse: most recent lead (received 2026-07-03, one day old — LIVE)

Source email: `New Tenant Lead matched your properties` from no.reply@leads.furnishedfinder.com.
FF housing-request emails carry a structured Stay Details table — parse confidence HIGH,
no inference needed:

```yaml
- id: 2026-07-03-ff-huzayfa-j
  received: 2026-07-03T15:40:53Z
  channel: furnished_finder
  type: housing_request            # area match, listing matched "Exact"
  unit_matched: "Ultra Clean Two-Story 2BR 1.5BA, In-Unit WD, Carport, Netflix"
  name: "Huzayfa J."
  contact: "hu***@gmail.com / 647-***-9867"
  desired_start: 2026-09-01
  desired_end: 2026-12-31          # 121 days ≈ 4 months
  end_tier: YELLOW                 # Dec 31 ∈ Dec 19–Jan 5 window — catches the Jan wave
  term_tier: "4-5mo (+5%)"
  occupants: 2
  pets: false
  facility: "Tesla"
  message: "We are both Tesla interns, non smokers, non drinkers. Just need a place to sleep"
  status: new                      # no reply sent — prototype is read-only
```

Notes:
- This lead is real and current (Sep 1 move-in, inquired yesterday). Under the workflow's
  2h SLA it would already have an I1 reply; recommended action logged in the session.
- Direct-message emails (e.g. 2026-05-04) confirm listing metadata is also embedded:
  listing title, city, and advertised rate ($3,800/mo observed) — usable for state seeding.
- Observed real listing names differ from the demo seed — units.yaml seeding should use
  the actual FF listing titles found in these emails.

## Remaining blockers (unchanged)

- furnishedfinder.com egress still denied by environment network policy (public-page and
  dashboard reads). Not needed for the lead-followup workflow — email carries the payload.
