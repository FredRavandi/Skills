# 4. Coupling and architecture

Assess how strongly modules depend on one another and how costly it is to change them together.

## Core heuristic
Strong dependencies should usually be close and cohesive. Dependencies across distant or independently changing modules should usually be weak and expressed through stable public contracts. A strong, distant dependency is a warning sign. Volatility matters: a distant dependency on a genuinely stable contract may be acceptable.

This is a qualitative heuristic, not a literal Boolean formula or a metric that can be computed reliably from imports alone.

## Inspect
- Deep imports into another module's private/internal directories or implementation files.
- Circular dependencies and dependency direction violations.
- Cross-layer access that bypasses a module's public API (for example, a presentation layer reaching directly into another component's database/repository layer).
- Repeated business rules, duplicated magic numbers or status codes across module boundaries.
- Shared mutable state, hidden global dependencies and excessive knowledge of another module's data model.
- Feature changes requiring coordinated edits across many distant modules (shotgun surgery).
- Public APIs that expose implementation details or unstable internal types.

## Architecture-rule tools
Layer and boundary rules are measurable, so prefer a tool to manual tracing. Run the repository's configured tool and report its result. If none is configured, trace manually and recommend one in the next actions. Examples:
- Python: import-linter (layers, forbidden and independence contracts).
- JS/TS: dependency-cruiser, or eslint-plugin-boundaries.
- Java/Kotlin: ArchUnit. .NET: NetArchTest or ArchUnitNET.
- Other stacks: the established architecture-rule or dependency-graph tool.

A passing tool shows only that the declared rules hold. Check that the rules describe the intended architecture and that recent changes did not loosen them (dimension 5).

## History analysis
Git history shows what static structure hides: files that keep changing, and files that change together. Run in bash (Git Bash or WSL on Windows) from the repository root. Exclude generated, vendored and lock files when reading the output.

```bash
# Hotspots: change frequency x size (lines, as a complexity proxy) over 12 months.
# Columns: score, commits, lines, file. Replace lines with a complexity metric
# from the project's configured tool where one exists.
git log --since=12.months --no-merges --name-only --format= | sed '/^$/d' | sort | uniq -c |
while read -r n f; do [ -f "$f" ] && printf '%s\t%s\t%s\n' "$n" "$(wc -l < "$f")" "$f"; done |
awk -F'\t' '{ printf "%d\t%d\t%d\t%s\n", $1*$2, $1, $2, $3 }' | sort -rn | head -30

# Change coupling: file pairs committed together. Columns: degree (shared commits as a
# percentage of the pair's average commits), shared commits, file A, file B.
# MAX skips bulk commits (formatting, renames); MIN drops pairs with too few shared commits.
git log --since=12.months --no-merges --name-only --format='@%h' |
awk -v MAX=30 -v MIN=5 '
  function flush(  i, j, a, b, t) {
    if (n > 1 && n <= MAX)
      for (i = 1; i <= n; i++) for (j = i + 1; j <= n; j++) {
        a = f[i]; b = f[j]; if (a > b) { t = a; a = b; b = t }
        pair[a "\t" b]++
      }
    n = 0
  }
  /^@/ { flush(); next }
  NF   { f[++n] = $0; rev[$0]++ }
  END  { flush()
         for (p in pair) if (pair[p] >= MIN) {
           split(p, x, "\t")
           printf "%d%%\t%d\t%s\n", 100 * pair[p] / ((rev[x[1]] + rev[x[2]]) / 2), pair[p], p
         } }' | sort -rn | head -30
```

Read the coupling output as follows:
- A high-degree pair inside one module, or a source file with its own test, is expected.
- A high-degree pair across modules, layers or services is change coupling. It is evidence of a hidden dependency or a misplaced boundary. Report it as Likely until the code confirms the shared knowledge (a duplicated rule, a shared data shape, a leaked internal type).
- Renamed files appear as two different paths, so history before a rename is split. Short histories and squash-merged workflows weaken the signal; state this in the report.

## Validate
- Trace import and call paths; inspect package exports, dependency graphs and actual ownership boundaries.
- Consider runtime registration, plugin systems, generated clients and framework conventions.
- Use the history analysis above to check whether modules change together in practice; do not infer volatility from names alone.
- Separate intentional orchestration from accidental coupling.

## Preferred direction
Use explicit public interfaces, adapters, events or APIs where they reduce knowledge across boundaries. Keep tightly collaborating implementation details cohesive and nearby. Avoid adding abstraction layers where they only relocate complexity.
