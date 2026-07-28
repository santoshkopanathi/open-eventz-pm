# Functional Test Scenarios — moved to the app repo

The consolidated, canonical **functional test scenarios** now live in the **app repo**, right
next to the tests they reference. This lets a CI job (`doc-parity`) verify that every scenario
marked `[A]` (automated) still names a test file that actually exists — so the test plan can't
silently drift from the real suite.

**Canonical file:** `06-app/TEST-SCENARIOS.md`
**On GitHub:** https://github.com/Imbillionaire/open-eventz/blob/master/TEST-SCENARIOS.md

The three previous versioned docs (base, v1.1 badges/filters, v1.2 price/analytics) were merged
into that file and are archived here with an `_archived` suffix for history:

- `functional-test-scenarios_archived.md`
- `functional-test-scenarios-v1.1-badges-filters_archived.md`
- `functional-test-scenarios-v1.2-price-analytics_archived.md`

*Consolidated and moved 2026-07-23.*
