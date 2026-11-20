# 2. Duplication and dead code

Find duplicated business rules and code that is genuinely unreachable or unused.

## Inspect
- Repeated blocks, near-duplicate functions, copied validation, calculations, state transitions, error handling and business rules.
- Duplicated literals with domain meaning: status codes, permission names, protocol identifiers, limits, magic numbers and configuration keys.
- Unused exports, functions, classes, dependencies, files, parameters and unreachable branches.
- Rewrite residue: obsolete implementations, compatibility layers no longer used, stale feature flags and abandoned tests.
- Incomplete refactors: a new implementation added while the old one stays in place and is still called from some paths, so the same rule now has two code paths. In a pull request, for each replaced function, class or module, check that callers moved to the new one and the old one was removed, or that keeping both is documented and intended.

## Rules of judgement
- Similar syntax is not always harmful duplication. Distinguish intentional parallel implementations, generated code, test fixtures and adapters from duplicated policy/business logic.
- Prefer shared domain rules where one change must apply consistently. Avoid extracting abstractions for coincidental similarity.
- Static dead-code tools can miss reflection, plugin loading, dependency injection, route registration, generated references, entry points and runtime-loaded modules. Verify before classifying as confirmed dead code.
- Mark dynamic/runtime-loaded code explicitly in configuration or documented manifests so future cleanup does not delete it accidentally.
- For legacy code, enforce **no new clones** and no increase in dead-code findings before attempting to eliminate the full backlog.

## Example tools
`jscpd` for clones (run it on changed files to catch copy-paste within a pull request); Vulture for Python; Knip for JS/TS. Validate findings against runtime conventions and project metadata.
