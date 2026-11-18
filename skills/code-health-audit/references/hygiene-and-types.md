# 1. Hygiene and types

Assess readable, consistently formatted, well-typed code and useful context.

## Inspect
- Untyped public inputs/outputs, implicit `any`, broad `object`/`dict` escape hatches, unchecked casts, unsafe null handling and type ignores.
- Missing or weak types at module/API boundaries. Prioritise contracts and externally supplied data; do not demand redundant annotations where inference is safe and idiomatic.
- Formatter/linter violations, inconsistent naming, unreachable branches and unused imports/variables.
- Comments that narrate obvious operations rather than explain why, constraints, trade-offs, invariants or non-obvious behaviour.
- Inline suppressions (`# noqa`, `type: ignore`, `// eslint-disable`, `@ts-ignore`, `@ts-nocheck`, etc.), file-level disables and broad exclusions.
- Schema/config validation gaps and unsafe parsing of external input.

## Rules of judgement
- Prefer strict types for public boundaries and meaningful domain types over blanket annotation noise.
- A comment length limit is not a universal quality rule. Flag comments for low information value, stale claims or missing rationale; do not mechanically reject a comment solely because it exceeds two lines.
- Suppressions require a narrow rule, narrow scope, explanation and review/expiry record. If policy bans inline suppressions, report any new occurrence as a gate violation; do not silently remove it.
- Check the codebase's actual strictness settings. Do not assume strict mode is enabled because a config file exists.

## Example tools
Python: Ruff, mypy, Pyright. TypeScript/JavaScript: `tsc`, ESLint or Oxlint, project formatter. Use configured equivalents for other stacks.
