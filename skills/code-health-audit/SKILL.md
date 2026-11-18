---
name: code-health-audit
description: Audit a repository for deterministic code health, maintainability, architecture, and quality-gate integrity. Use when reviewing a codebase or pull request for tech debt, code quality, complexity, duplication, dead code, coupling, or weakened tests/lint/CI gates; when establishing quality baselines or ratchets; or when designing enforcement for AI coding agents. Read-only by default; never weaken gates or modify production code during an audit.
---

# Code Health Audit

## Mission

Perform an evidence-based audit of code health and of the mechanisms that protect it. Prefer deterministic checks over subjective instructions: **if a property can be measured, enforce it with a tool rather than asking an agent to remember it.** A green build is necessary evidence, not proof of sound design or a correct solution.

Two modes:

1. **Audit mode (default):** inspect the repository, run safe existing checks, compare against the trusted baseline, and report findings. Do not edit application code, tests, CI, hooks, thresholds or configuration.
2. **Gate-design mode (only when explicitly requested):** propose or implement a quality-gate plan. Obtain approval before changing enforcement, configuration, hooks, CI, branch protection or CODEOWNERS.

**Not for:** hunting functional bugs in a single diff, a dedicated security/penetration review, or performance profiling. Note such issues if you meet them, but use a purpose-built review for them.

## Non-negotiable rules

- Be evidence-led. Every Confirmed finding needs a file path and line range, command output, test result or reproducible dependency path. Do not invent findings.
- Label every finding **Confirmed**, **Likely** or **Conjectural** (defined under Severity and confidence).
- Distinguish objective violations from design judgement. Explain the rule and impact; do not present taste as fact.
- Do not fix findings during an audit. Do not make unrelated edits.
- Never silence a failing check, relax a threshold, remove or skip a test, broaden an exclusion, or change the baseline to obtain a pass.
- Do not assume a symbol is dead merely because static search finds no reference. Check reflection, plugin discovery, dependency injection, generated code, entry-point metadata, framework conventions, dynamic imports, configuration references and runtime registration.
- Do not impose a new toolchain blindly. Use tools already configured where possible. Recommend new tools only with a clear benefit and adoption cost.
- Do not fail a legacy repository merely because pre-existing debt exists. Establish a baseline and gate regressions/new violations.
- Treat secrets, credentials, private data and production systems carefully. Do not print secret values in the report. Do not run tests or scripts that touch external services, databases or production without confirmation.

## Audit workflow

### 1. Establish scope and trusted state

- Identify repository root, languages, frameworks, package managers, build system, test framework, and generated/vendor directories.
- Inspect `git status`, current branch, merge base with the target branch, and recent changes. Do not discard or overwrite user changes.
- Identify the trusted policy source: normally the protected default branch, a protected CI workflow, or an explicitly supplied policy. A pull request's modified configuration is not automatically trusted.
- Classify the request: whole-repository, changed-files, pull-request, or quality-gate design.
- For large repositories, prioritise hotspots (files that are both frequently changed and complex) rather than sampling at random. Change frequency: `git log --since=12.months --name-only --format= | sed '/^$/d' | sort | uniq -c | sort -rn | head -30`. State the sampling strategy in the report.
- Record the exact scope and any unavailable checks.

### 2. Discover existing rules and checks

Inspect, as applicable: formatter, linter, type checker, compiler and build configuration; test commands, coverage and mutation-test rules; complexity and duplication thresholds; static-analysis and security tooling; CI workflows, pre-commit hooks, agent hooks and scripts; ignore/exclusion files, suppression comments and generated-code declarations; CODEOWNERS, branch-protection documentation and policy files; baseline files and historical quality reports.

Run existing safe checks first. Derive commands from project manifests, scripts, docs and CI; do not invent them. If a command may modify files, use its check/dry-run mode or do not run it. Record tool name/version, command, scope, exit code and relevant output for every check.

### 3. Assess the six dimensions

Read each reference file before assessing its dimension. Adapt the checks to the language and framework.

| # | Dimension | Reference | Core question |
|---|---|---|---|
| 1 | Hygiene & types | `references/hygiene-and-types.md` | Are boundaries typed, code consistently formatted, suppressions justified? |
| 2 | Duplication & dead code | `references/duplication-and-dead-code.md` | Are business rules duplicated? Is unused code genuinely unused? |
| 3 | Code smells | `references/code-smells.md` | Which structures raise the cost of future change? |
| 4 | Coupling & architecture | `references/coupling-and-architecture.md` | Do dependencies respect boundaries, direction and volatility? |
| 5 | Gate integrity | `references/gate-integrity.md` | Were checks, tests, thresholds or baselines weakened relative to the trusted base? |
| 6 | Quality-gate engineering | `references/quality-gate-engineering.md` | Do existing gates run in the right loop, block on failure, and cover the most frequent review failures? |

For dimension 5, run the deterministic diff checks in `references/gate-integrity.md` when a trusted base exists. A gate change is not automatically malicious: classify intent and require explicit review where policy changed.

For dimension 4, trace representative call paths. Do not infer coupling solely from directory layout or import depth.

In audit mode, dimension 6 evaluates the gates that exist. Designing new gates belongs to gate-design mode.

### 4. Report, do not repair

Fill in `templates/audit-report.md`. Return the report in the conversation; write it to a file only when the user asks, and never inside the audited source tree unless told to. Prioritise by impact and confidence. If no issues are found, state what was checked and what was not; do not claim the codebase is defect-free.

## Tool selection

Use the repository's existing ecosystem first. Examples, not mandatory dependencies:

- Python: Ruff, mypy or Pyright, pytest, coverage, Vulture, jscpd; Pylint/SonarQube where justified.
- TypeScript/JavaScript: `tsc`, ESLint or Oxlint, the project's formatter, Knip, jscpd, SonarJS.
- Other languages: the native compiler, formatter, linter, test runner and established static-analysis tools.

Do not install tools or change lockfiles during audit mode. Do not treat tool output as infallible; validate high-impact findings and account for dynamic behaviour.

## Severity and confidence

Severity:
- **Critical:** credible path to major security, data-integrity, availability or safety impact.
- **High:** likely production failure, serious architectural boundary violation, or deliberate/consequential weakening of required controls.
- **Medium:** material maintainability, reliability or testability debt with plausible future cost.
- **Low:** localised hygiene issue with limited impact.
- **Info:** observation or improvement opportunity, not a defect.

Confidence:
- **Confirmed:** directly demonstrated by deterministic output, code path, diff or reproducible test.
- **Likely:** strong evidence, but runtime or contextual validation is incomplete.
- **Conjectural:** plausible concern requiring investigation; do not count as a confirmed violation.

Do not inflate severity because a pattern is on a checklist. State impact, reach, likelihood and evidence.

## Completion criteria

An audit is complete only when every section of `templates/audit-report.md` is filled in or explicitly marked not applicable with a reason. In particular: scope and trusted base, all six dimensions marked assessed or not assessed, checks run with versions and exit codes, checks not run and why, baseline/new-code treatment, findings with evidence and confidence, gate-integrity assessment, and prioritised next actions.

## Gate-design mode

Use `references/quality-gate-engineering.md` (three loops, baselines, ratcheting, rollout) and the protection model in `references/gate-integrity.md`. First present the proposed policy and the files to be changed. Implement only after approval, then test that each gate fails when it should, and document how the gates are protected. A skill instruction is not a security boundary: CI and repository permissions are.
