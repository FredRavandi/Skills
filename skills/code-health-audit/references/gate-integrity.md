# 5. Gate integrity and anti-gaming

Check whether quality controls were bypassed, weakened or removed, intentionally or otherwise.

## Diff checks against trusted base
- Added/expanded lint, type-check, coverage, security or test suppressions.
- Raised complexity/duplication thresholds or relaxed strict compiler/linter settings.
- New ignore patterns, exclusions, skipped paths, disabled rules or conditional bypasses.
- Deleted, weakened, renamed or skipped tests; reduced assertions; removed test data or fixtures needed to expose a failure.
- Modified package scripts, Makefiles, task runners, CI commands, hooks or required-check definitions to omit checks or mask exit codes.
- Replaced failing checks with non-blocking/continue-on-error steps, allowed failures, neutral statuses or success fallbacks.
- Re-generated or edited baselines to absorb new violations without an independently reviewed reason.
- Changes to CODEOWNERS, branch protection definitions, policy files or trusted workflow references.

## Deterministic triage commands
Run from the repository root. Replace `origin/main` with the trusted target branch. These are triage signals: every hit needs review, and no hits does not prove integrity (for example, exclusions added through config files or CI settings outside the repository).

```bash
BASE=$(git merge-base origin/main HEAD)
git diff --stat "$BASE"...HEAD

# Deleted or renamed test files
git diff --name-status --diff-filter=DR "$BASE"...HEAD -- '*test*' '*spec*'

# Changed gate/policy files (extend the list for the stack)
git diff --name-only "$BASE"...HEAD | grep -E '(^|/)(\.github/workflows/|\.gitlab-ci|Jenkinsfile|azure-pipelines|\.pre-commit-config|\.husky/|CODEOWNERS|Makefile|package\.json|pyproject\.toml|setup\.cfg|tox\.ini|\.eslintrc|eslint\.config|tsconfig|ruff\.toml|mypy\.ini|\.coveragerc|jest\.config|vitest\.config|\.claude/settings)'

# Added suppressions, skips and masked failures (file headers show where each hit is)
P='noqa|type: *ignore|pyright: *ignore|pylint: *disable|pragma: *no cover|eslint-disable|@ts-(ignore|nocheck|expect-error)|NOSONAR|nolint|@SuppressWarnings|\.(skip|only)\(|\bx(it|describe)\(|pytest\.mark\.(skip|xfail)|@Disabled|@Ignore|continue-on-error: *true|\|\| *true|--no-verify|allow_failure: *true'
git diff -U0 "$BASE"...HEAD | grep -E "^\+\+\+ |^\+.*($P)" | grep -B1 -E '^\+[^+]'

# Assertion balance in tests (removed vs added lines)
git diff "$BASE"...HEAD -- '*test*' '*spec*' | grep -cE '^-[^-].*\b(assert|expect\()'
git diff "$BASE"...HEAD -- '*test*' '*spec*' | grep -cE '^\+[^+].*\b(assert|expect\()'
```

## Interpretation
- Compare with the trusted target branch, not only with the candidate branch's own configuration.
- A test deletion or threshold change is not automatically malicious. Determine whether it is justified, whether equivalent coverage exists, and whether the policy owner approved it.
- Report both the exact diff and its effect on enforcement. Do not infer intent without evidence.
- Check whether the CI workflow actually executes the required gate, whether it ran rather than being skipped or reported neutral, and whether branch protection requires its result.

## Protection model
- A skill or prompt instruction is not a security boundary. Repository permissions, required CI checks and human review are.
- Protect rules, thresholds, baselines, hooks and CI policy with repository permissions and human review (CODEOWNERS and branch protection or equivalent).
- Evaluate PRs with trusted policy from the target branch or a protected reusable workflow; review proposed policy changes separately.
- Make required checks non-optional in branch protection.
- Use local agent/editor hooks for fast feedback and blocking where the host supports it, but treat CI as the authoritative shared gate.
- Log gate changes and exceptions. Alert on new suppressions, exclusions, threshold increases, deleted tests and modified check commands.
- Ensure a check cannot edit or redefine the policy used to judge that same check. Avoid a single mutable script being both the policy and the judge of its own modifications.
