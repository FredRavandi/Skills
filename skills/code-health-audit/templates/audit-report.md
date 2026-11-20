# Code Health Audit

## Executive summary
- Overall assessment:
- Highest-priority risk:
- Gate integrity: Intact / At risk / Weakened / Not assessed
- Test effectiveness: Demonstrated / Weak / Unknown / Not assessed

## Scope and trust basis
- Repository:
- Commit / branch:
- Audit scope: Whole repository / Changed files / Pull request / Gate design
- Sampling strategy (if not exhaustive):
- Trusted base / policy source:
- Languages and major frameworks:

## Dimension coverage
| # | Dimension | Status | Findings (C/H/M/L/I) | Notes |
|---|---|---|---|---|
| 1 | Hygiene & types | Assessed / Partial / Not assessed | | |
| 2 | Duplication & dead code | | | |
| 3 | Code smells | | | |
| 4 | Coupling & architecture | | | |
| 5 | Gate integrity | | | |
| 6 | Quality-gate engineering | | | |
| 7 | Test effectiveness | | | |

## Deterministic checks
| Check | Tool/version | Command | Scope | Exit code | Result | Notes |
|---|---|---|---|---|---|---|

## Baseline and new-code treatment
- Baseline source (file, commit, tool versions) or "none":
- Comparison method: Changed lines / New code / Repository-wide counts / None
- Existing debt excluded from pass/fail:
- New or worsened violations:

## Findings
For each finding:

### [ID] [Severity] Short title
- **Confidence:** Confirmed / Likely / Conjectural
- **Category:** Hygiene & types / Duplication & dead code / Code smells / Coupling & architecture / Gate integrity / Quality-gate engineering / Test effectiveness
- **Evidence:** `path/to/file.ext:line-line`, diff, command output or reproducible path
- **Rule:** The measurable standard or design principle
- **Impact:** Why it matters and likely reach
- **Recommendation:** Smallest effective corrective action
- **Baseline status:** Existing / New / Worsened / Not applicable

## Gate-integrity review
- Suppressions and exclusions:
- Threshold/configuration changes:
- Test additions, removals or weakening:
- CI/hooks/scripts changes:
- Agent instruction, permission and hook changes:
- Baseline changes:
- New dependencies (each checked for use, name, pinning and registry):
- Preventive controls on rule files (review protection, agent-side protection):
- Human approvals required:

## Verification loops
| Loop | Present? | Blocks on failure? | Evidence |
|---|---|---|---|
| Inner (after each edit) | | | |
| Middle (before work is declared complete) | | | |
| Adversarial review before release | | | |
| Outer (PR/CI) | | | |
| Production feedback | | | |
| Observation (periodic harness review) | | | |

## Test effectiveness
- Mutation testing: tool/version, scope, score, date of report; or "not configured" / "not run" and why:
- Tautological or assertion-free tests found:
- Coverage figure, if available (execution only, not proof of checking):

## Checks not run
List each check, the reason, and what evidence remains unavailable.

## Prioritised next actions
1. Immediate:
2. Next:
3. Longer-term ratchet:

## Limitations
State what the audit cannot establish. A passing audit is not proof of correctness, security, fitness for purpose or sound architecture.
