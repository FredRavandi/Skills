# 5. Gate integrity and anti-gaming

Check whether quality controls were bypassed, weakened or removed, intentionally or otherwise, and whether the change added dependencies nobody verified.

## Diff checks against trusted base
- Added/expanded lint, type-check, coverage, security or test suppressions.
- Raised complexity/duplication thresholds or relaxed strict compiler/linter settings.
- New ignore patterns, exclusions, skipped paths, disabled rules or conditional bypasses.
- Deleted, weakened, renamed or skipped tests; reduced assertions; removed test data or fixtures needed to expose a failure.
- Modified package scripts, Makefiles, task runners, CI commands, hooks or required-check definitions to omit checks or mask exit codes.
- Replaced failing checks with non-blocking/continue-on-error steps, allowed failures, neutral statuses or success fallbacks.
- Re-generated or edited baselines to absorb new violations without an independently reviewed reason.
- Changes to CODEOWNERS, branch protection definitions, policy files or trusted workflow references.
- Changes to agent instruction, permission or hook files (`CLAUDE.md`, `AGENTS.md`, `.claude/`, `.cursor/` and equivalents) that loosen rules or let the agent edit protected files.
- New or re-pinned dependencies, and lockfile entries that resolve from a different registry (see New dependencies).

## Deterministic triage commands
Run in bash (Git Bash or WSL on Windows), not PowerShell, from the repository root. Replace `origin/main` with the trusted target branch. These are triage signals: every hit needs review, and no hits does not prove integrity (for example, exclusions added through config files or CI settings outside the repository).

```bash
BASE=$(git merge-base origin/main HEAD)
git diff --stat "$BASE"...HEAD

# Deleted or renamed test files
git diff --name-status --diff-filter=DR "$BASE"...HEAD -- '*test*' '*spec*'

# Changed gate, policy, agent-instruction and baseline files (extend the lists for the stack)
G='\.github/workflows/|\.gitlab-ci|Jenkinsfile|azure-pipelines|\.pre-commit-config|\.husky/|CODEOWNERS|Makefile|package\.json|pyproject\.toml|setup\.cfg|tox\.ini|pytest\.ini|\.flake8|\.eslintrc|eslint\.config|biome\.json|tsconfig|ruff\.toml|mypy\.ini|\.coveragerc|jest\.config|vitest\.config|stryker\.conf|\.importlinter|\.dependency-cruiser|pom\.xml|build\.gradle|sonar-project\.properties|\.golangci'
A='CLAUDE\.md|AGENTS\.md|\.claude/|\.cursor/|\.cursorrules|copilot-instructions\.md|[^/]*baseline[^/]*'
git diff --name-only "$BASE"...HEAD | grep -E "(^|/)($G|$A)"

# Added suppressions, skips and masked failures (file headers show where each hit is)
P='noqa|type: *ignore|pyright: *ignore|pylint: *disable|pragma: *no cover|eslint-disable|@ts-(ignore|nocheck|expect-error)|NOSONAR|nolint|@SuppressWarnings|\.(skip|only)\(|\bx(it|describe)\(|pytest\.mark\.(skip|xfail)|@Disabled|@Ignore|continue-on-error: *true|\|\| *true|--no-verify|allow_failure: *true'
git diff -U0 "$BASE"...HEAD | grep -E "^\+\+\+ |^\+.*($P)" | grep -B1 -E '^\+[^+]'

# Assertion balance in tests (removed vs added lines). A rising count is not evidence of
# stronger tests: check added assertions with the tautology commands in test-effectiveness.md.
git diff "$BASE"...HEAD -- '*test*' '*spec*' | grep -cE '^-[^-].*\b(assert|expect\()'
git diff "$BASE"...HEAD -- '*test*' '*spec*' | grep -cE '^\+[^+].*\b(assert|expect\()'
```

## New dependencies
Agents can invent package names. Attackers register commonly hallucinated or mistyped names ("slopsquatting" and typosquatting), and publish public packages that shadow private names (dependency confusion). Treat every new dependency as unverified until checked.

```bash
# Added or changed manifest lines: list each new or re-pinned package
git diff -U0 "$BASE"...HEAD -- '*package.json' '*requirements*.txt' '*requirements*.in' '*pyproject.toml' '*Pipfile' '*setup.cfg' '*setup.py' '*go.mod' '*Cargo.toml' '*Gemfile' '*.gemspec' '*pom.xml' '*build.gradle' '*build.gradle.kts' '*.csproj' '*Directory.Packages.props' '*composer.json' | grep -E '^(\+\+\+ |\+[^+])' | grep -B1 -E '^\+[^+]'

# Registries that newly added lockfile entries resolve from
git diff -U0 "$BASE"...HEAD -- '*package-lock.json' '*npm-shrinkwrap.json' '*yarn.lock' '*pnpm-lock.yaml' '*poetry.lock' '*uv.lock' '*Pipfile.lock' '*Cargo.lock' '*packages.lock.json' '*go.sum' | grep -E '^\+[^+]' | grep -oE 'https?://[^/" ]+' | sort | uniq -c
```

For each new direct dependency, establish from the repository first:
- The code actually imports or uses it, and the import name matches the package name declared.
- It is not a near-miss of a well-known package: one or two characters off, swapped `-`/`_`/`.`, a missing or different scope, or a plausible-sounding variant.
- The lockfile pins the version with an integrity hash, and the lockfile changed consistently with the manifest.
- Internal or private packages resolve from the intended private index, not a public registry.
- Install-time scripts (for example npm `preinstall`/`postinstall`) are expected.

Registry lookups (publish date, maintainer, download history, linked source repository) are external calls. Ask the user before making them, and query public registries only for packages meant to come from them. **Never query a public registry for an internal or private package name.** Doing so discloses the name, which is the first step of a dependency-confusion attack.

Severity: a package that does not exist, resolves from the wrong registry, or closely imitates a popular name is High or Critical until disproved. A legitimate but unneeded dependency is Low or Medium.

## Interpretation
- Compare with the trusted target branch, not only with the candidate branch's own configuration.
- A test deletion or threshold change is not automatically malicious. Determine whether it is justified, whether equivalent coverage exists, and whether the policy owner approved it.
- Report both the exact diff and its effect on enforcement. Do not infer intent without evidence.
- Check whether the CI workflow actually executes the required gate, whether it ran rather than being skipped or reported neutral, and whether branch protection requires its result.

## Preventive controls (assess in audit mode)
The diff checks find weakening after it happens. Also report whether anything stops it happening:
- Do CODEOWNERS (or equivalent) and branch protection require human review for rule files, thresholds, baselines, hooks, CI policy and dependency manifests? Record "not visible from the repository" when settings live only in the hosting platform.
- Does the agent configuration deny edits to those paths, for example agent permission deny rules or a hook on file-edit tools? Read the host's own configuration and documentation; do not assume hook names or exit codes.
- An agent with shell access can bypass rules that only match file-edit tools (for example with `sed -i` or `git checkout`). Report agent-side protection as a speed bump. Its absence alone is Low or Info. Missing review protection on rule files in a repository where agents commit is Medium or High.

## Protection model
- A skill or prompt instruction is not a security boundary. Repository permissions, required CI checks and human review are.
- Protect rules, thresholds, baselines, hooks, CI policy and dependency manifests with repository permissions and human review (CODEOWNERS and branch protection or equivalent).
- Evaluate PRs with trusted policy from the target branch or a protected reusable workflow; review proposed policy changes separately.
- Make required checks non-optional in branch protection.
- Use local agent/editor hooks for fast feedback and blocking where the host supports it, but treat CI as the authoritative shared gate.
- Log gate changes and exceptions. Alert on new suppressions, exclusions, threshold increases, deleted tests, modified check commands and new dependencies.
- Ensure a check cannot edit or redefine the policy used to judge that same check. Avoid a single mutable script being both the policy and the judge of its own modifications.
