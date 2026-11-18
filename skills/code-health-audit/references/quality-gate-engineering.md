# 6. Quality-gate engineering

Turn recurring review feedback into enforceable, progressively stricter repository policy.

## In audit mode
Evaluate the gates that already exist; do not design or add new ones.
- List each check, the loop it runs in (inner/middle/outer), and whether it blocks on failure or only reports.
- Identify checks that are configured but never run, run but never block, or run in a loop too late or too expensive for their purpose.
- Compare the most frequent review findings (from this audit, PR history or the user) with the automated checks. Report each measurable, recurring failure that has no gate as an Info or Medium finding.
- Note whether a baseline exists, whether it is protected, and whether it is ratcheting or static.

## Three loops
1. **Inner loop (after each edit):** formatter, linter, type checker, targeted tests and cheap banned-pattern/suppression checks. Fast, exact diagnostics so the agent can correct its own change; no broad or flaky scans.
2. **Middle loop (before work is declared complete):** relevant tests, changed-code complexity and clone detection, dead-code analysis with dynamic-use safeguards, and architecture checks.
3. **Outer loop (PR/CI):** complete required test suite, trusted-policy evaluation, security/dependency checks, test-removal and gate-integrity checks, and baseline comparison.

Run each check in the earliest loop where it is reliable and affordable. Expensive or noisy checks belong later unless risk justifies earlier execution.

## Deterministic backpressure
- Prefer executable checks with stable inputs, pinned versions and machine-readable results.
- A failing gate must return a non-zero status and actionable diagnostics. Do not swallow errors or convert failures into success.
- Wire checks into agent/editor hooks for fast feedback and into required CI for shared enforcement.
- Host-specific hook exit codes and semantics vary. Inspect the installed tool's current documentation/configuration; never assume an exit code or hook event is portable.

## Baseline and ratchet
- Capture a reproducible baseline on a known-good commit with pinned tool versions and explicit scope.
- Fail on new violations or worsening counts. Prefer changed-lines/new-code comparison where reliable; otherwise compare repository-wide counts and fail only when count or severity worsens.
- Keep baseline changes reviewable and protected. A feature branch must not regenerate or redefine its own pass criteria.
- Ratchet one metric (or a small number) at a time: baseline, prevent regression, then tighten once the backlog shrinks.
- Track exceptions with owner, rationale, scope, expiry/review date and issue reference. Keep exceptions narrow and visible.
- A baseline is not approval of every existing defect.
- Convert a repeated human review comment into a measurable rule only when the rule captures the underlying risk without creating perverse incentives.

## Goodhart safeguards
Metrics can be gamed. Pair counts with review of intent and impact. Watch for artificial function splitting, meaningless wrappers, exclusion growth, suppression growth, assertion removal and complexity-threshold changes. Never optimise a score at the expense of correctness, clarity or system understanding.

## Minimum practical rollout
1. Identify the most frequent, measurable review failure.
2. Add one fast local gate and one required CI gate using the same trusted rule.
3. Baseline legacy violations and block new ones.
4. Protect the rule/configuration and require human review for changes.
5. Review false positives and developer friction, then ratchet gradually.
