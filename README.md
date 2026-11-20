# Skills

A collection of [Agent Skills](https://agentskills.io) for Claude Code and other skill-compatible agents. Each skill is a self-contained folder under [`skills/`](skills/) with a `SKILL.md` entry point and any reference files it needs.

## Available skills

| Skill | What it does |
|---|---|
| [code-health-audit](skills/code-health-audit/) | Read-only, evidence-based audit of code health and quality-gate integrity across seven dimensions: hygiene & types, duplication & dead code, code smells, coupling & architecture (including change coupling from git history), gate integrity (weakened tests, new suppressions, masked CI failures, unverified new dependencies), quality-gate engineering, and test effectiveness (tautological tests, mutation testing). Includes bash triage commands and a report template. |

## Install

Clone the repository once, then copy (or link) the skill folders you want into a skills directory:

- `~/.claude/skills/` for all your projects
- `<project>/.claude/skills/` for a single project

**macOS / Linux**

```bash
git clone https://github.com/FredRavandi/Skills.git
cp -r Skills/skills/code-health-audit ~/.claude/skills/
```

**Windows (PowerShell)**

```powershell
git clone https://github.com/FredRavandi/Skills.git
Copy-Item -Recurse Skills\skills\code-health-audit "$HOME\.claude\skills\"
```

To pick up updates with `git pull` instead of re-copying, link the folder rather than copying it. Use `ln -s` on macOS/Linux, or `New-Item -ItemType Junction` on Windows.

Claude Code discovers skills automatically. Only each skill's name and description load up front. The body of `SKILL.md` loads when the skill is triggered, and the reference files load only when needed.

## Use

Ask in natural language. For example, "audit this repo's code health" or "review this PR for weakened quality gates against main". You can also invoke a skill by name, for example `/code-health-audit`.

## Repository layout

```
skills/
  <skill-name>/
    SKILL.md        # entry point (frontmatter: name, description)
    references/     # optional, loaded on demand
    templates/      # optional
```

## License

All skills in this repository are released under the [MIT License](LICENSE).
