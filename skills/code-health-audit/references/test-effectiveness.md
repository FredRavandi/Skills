# 7. Test effectiveness

Assess whether the tests would fail if the code were wrong. Passing tests, line coverage and assertion counts do not establish this. Tests written by the same agent that wrote the code often restate the implementation instead of checking it.

## Inspect
- Tautological tests: assertions that compare a value with itself or with a constant (`assert x == x`, `assert True`, `expect(true).toBe(true)`).
- Tests that assert what a mock was configured to return, or compute the expected value with the same code or formula under test.
- Tests whose only check is "no exception", "not None", "truthy", "was called" or an unreviewed snapshot.
- Tests with no assertion, or with assertions only inside branches or loops that may not run.
- Over-mocking: the unit under test, or the collaborator that holds the business rule, is mocked away.
- Boundary gaps: comparisons, ranges, limits, empty/null/maximum inputs and off-by-one cases without a test at the boundary and on each side of it.
- Snapshot or golden files regenerated in the same change as the code they cover.
- High line coverage with a low mutation score: the tests execute the code without checking its results.

## Mutation testing
Mutation testing makes small changes to the code (for example `>` to `>=`, `+` to `-`, a negated condition or a removed call) and runs the tests against each change. A mutant that survives shows behaviour the tests do not check.

- **Audit mode:** check whether a mutation tool is configured, whether CI runs it, whether its result blocks a merge, and what the latest report shows. Report the score, its scope and surviving mutants in high-risk code from existing reports.
- **Do not run mutation testing in audit mode without user confirmation.** Runs are slow. Some tools write mutated files into the working tree during a run, and an interrupted run can leave them there. If the user confirms: start from a clean working tree or a disposable copy, scope the run to changed or hotspot files, record tool version, scope, score and duration, and confirm `git status` is clean afterwards.
- An equivalent mutant (a change that does not alter behaviour) is not a test gap. Classify surviving mutants before reporting them.
- Example tools: mutmut or Cosmic Ray (Python), StrykerJS (JS/TS), PIT (Java/JVM), Stryker.NET (.NET), cargo-mutants (Rust). Use the repository's configured tool if one exists.

## Deterministic triage commands
Run in bash (Git Bash or WSL on Windows; the commands use a bash array), from the repository root. Set `BASE` as in `gate-integrity.md`. These are triage signals: read every hit before reporting it, and no hits does not show the tests are effective. The patterns cover common Python, JS/TS and JUnit forms; extend them for the stack.

```bash
TAUT=(
  '\bassert +(True|1)[[:space:]]*$|\bassert(True|That)?\((true|True|1)\)|expect\((true|1)\)\.(toBe|toEqual|toBeTruthy)\b'
  '\bassert +([A-Za-z_][A-Za-z0-9_.]*) *== *\1[[:space:]]*$'
  'expect\(([A-Za-z_][A-Za-z0-9_.]*)\)\.to(Be|Equal|StrictEqual)\(\1\)'
  '\bassert(Equal|Equals)\(([A-Za-z_][A-Za-z0-9_.]*), *\2\)'
)
# Whole repository: constant and self-comparing assertions in tests at HEAD (path:line output)
for p in "${TAUT[@]}"; do git grep -nE "$p" HEAD -- '*test*' '*spec*'; done

# Pull request: the same patterns in added test lines only (file headers show where each hit is)
for p in "${TAUT[@]}"; do
  git diff -U0 "$BASE"...HEAD -- '*test*' '*spec*' | grep -E '^(\+\+\+ |\+[^+])' | grep -E "^\+\+\+ |$p" | grep -B1 -E '^\+[^+]'
done
```

Each self-comparison pattern uses a backreference, so it must stay a separate pattern. Linters catch more of these cheaply when configured, for example ESLint `no-self-compare`, `jest/expect-expect` or its Vitest equivalent, and Ruff `PLR0124` (comparison with itself).

## Rules of judgement
- Classify a tautological or assertion-free test as Confirmed only after reading it.
- More assertions or more coverage is not evidence of stronger tests. Judge whether each test can fail for the defect it claims to guard against.
- Look for an independent oracle: a specification, known example values, property-based tests, or fixtures taken from real data. A test whose expected value was produced by running the code under test has none.
- Severity depends on what the weak test guards. False assurance over money, safety, access control or data integrity is Medium or High; over trivial code it is Low.

## Gate policy
- Inner loop: lint rules for self-comparison and assertion-free tests.
- Middle or outer loop: mutation testing on changed code, using incremental or changed-files mode where the tool supports it. Baseline the mutation score per module and fail when changed code adds surviving mutants on covered lines or lowers the score. Ratchet upwards.
- Protect mutation configuration like any other threshold: excluded files, disabled mutators, timeouts and score thresholds.
