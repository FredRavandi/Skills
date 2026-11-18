# 3. Code smells

Identify structures that increase the cost and risk of future changes. A smell is a prompt for investigation, not automatic proof of a defect.

## Inspect
- Long parameter lists and repeated groups of related primitive parameters.
- Primitive obsession: raw strings/numbers/booleans representing domain concepts, identifiers, units, money, states or constrained values.
- High cyclomatic/cognitive complexity, deep nesting, long functions and excessive branching.
- God classes/modules, feature envy, shotgun surgery, inappropriate intimacy and excessive responsibilities.
- Unused parameters, duplicated strings, boolean flags that select unrelated behaviours, hidden side effects and inconsistent error handling.

## Better design signals
- Use cohesive value objects/domain types for meaningful concepts (for example, validated identifiers, units, money or configuration). In Python, immutable/frozen dataclasses may fit; in TypeScript, branded types or validated schemas may fit. Do not introduce wrappers without domain value.
- Split responsibilities along stable reasons to change, not arbitrary line counts.
- Reduce complexity by clarifying domain decisions and control flow, not merely extracting tiny functions or gaming a metric.
- Treat complexity thresholds as policy. Never increase them merely to pass a change; threshold changes require separate human review.

## Gate policy
For existing repositories, baseline current findings and fail on new issues or worsening changed-code metrics. Review the highest-impact smells in pull requests even if the tool reports no threshold breach.
