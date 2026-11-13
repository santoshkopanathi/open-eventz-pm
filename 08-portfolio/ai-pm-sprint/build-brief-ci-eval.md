# Build brief — automate the Open Eventz eval in CI

*Paste this into the build thread. Written 2026-10-21 from a read of `06-app/eval/`.*

## Goal
Turn the manual 64-event golden-set eval into a CI check that runs on every prompt, model or classifier change, and blocks the change if any wrongly-free label appears.

## What exists today
- `eval/_parsed.json`: 68 events, 64 hand-labelled (kid yes/no, price free/paid/unknown, notes). **Git-ignored.**
- `eval/price-golden-worksheet.md`: the labelling worksheet, with a copy of each event's description (lengths up to ~1,000 characters; check whether they were trimmed).
- `eval/run-v2.js`, `eval/run-v3.js`: run a model over the set. Each has **its own pasted copy of the prompt**, and reads event descriptions **live from Supabase**.
- `eval/score.js`: scores a results file against four gates (confidence varies, calibration, wrongly-free = 0, kid-relevance).
- `src/lib/price-calibration.llm.ts` (`npm run calibrate:price`): a separate small calibration run against the production classifier.

## Problems to fix first
1. **Inputs aren't frozen.** Play Frisco events are purged nightly, so many golden-set events are probably gone from Supabase. Re-running today would silently test fewer events.
2. **The eval tests a copy of the prompt,** not the production classifier (`inferPlayFriscoEvent` in `src/lib/age-inference.ts`).
3. **Labels are git-ignored,** so CI can't see them. Decision for the owner: commit labels plus event text (public civic data), or keep them private.

## Steps
1. Freeze the inputs: export every golden-set event still in Supabase (title, description, source, registration flag) into a committed fixture; recover the rest from the worksheet and flag any that may be trimmed. Report how many of the 64 survive intact.
2. Run the eval through the production classifier and the same price resolution the pipeline uses (`resolvePriceClass`), not a pasted prompt.
3. Score automatically. **Blocking:** wrongly-free = 0. **Flagged:** over-claims rising beyond a small agreed tolerance vs. the stored baseline; any malformed answers.
4. Add a GitHub Actions job that runs only when `src/lib/age-inference.ts`, `src/lib/price.ts`, the model ID or the eval fixtures change. API key from GitHub secrets.
5. Record cost (tokens in/out) and run time per run in the job summary.
6. Update `EVALS.md` and `eval/RUN-LOG.md`.

## Done when
- One command runs the eval locally against frozen inputs and prints the gate results.
- The CI job runs on a prompt change and fails on a seeded wrongly-free case.
- Cost per run is recorded.

## Out of scope
Changing the prompt or the model. That's a separate decision, gated by this eval.
