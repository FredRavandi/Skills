# 4. Coupling and architecture

Assess how strongly modules depend on one another and how costly it is to change them together.

## Core heuristic
Strong dependencies should usually be close and cohesive. Dependencies across distant or independently changing modules should usually be weak and expressed through stable public contracts. A strong, distant dependency is a warning sign. Volatility matters: a distant dependency on a genuinely stable contract may be acceptable.

This is a qualitative heuristic, not a literal Boolean formula or a metric that can be computed reliably from imports alone.

## Inspect
- Deep imports into another module's private/internal directories or implementation files.
- Circular dependencies and dependency direction violations.
- Cross-layer access that bypasses a module's public API (for example, reaching directly into another component's database/repository layer).
- Repeated business rules, duplicated magic numbers or status codes across module boundaries.
- Shared mutable state, hidden global dependencies and excessive knowledge of another module's data model.
- Feature changes requiring coordinated edits across many distant modules (shotgun surgery).
- Public APIs that expose implementation details or unstable internal types.

## Validate
- Trace import and call paths; inspect package exports, dependency graphs and actual ownership boundaries.
- Consider runtime registration, plugin systems, generated clients and framework conventions.
- Identify whether the modules change together in practice using history where available; do not infer volatility from names alone.
- Separate intentional orchestration from accidental coupling.

## Preferred direction
Use explicit public interfaces, adapters, events or APIs where they reduce knowledge across boundaries. Keep tightly collaborating implementation details cohesive and nearby. Avoid adding abstraction layers where they only relocate complexity.
