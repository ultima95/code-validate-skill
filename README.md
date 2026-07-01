# code-validate

An **evidence-based code review skill** for AI coding agents. It reviews a diff, branch, or pull request against **only what the repository proves** — a missed issue is preferable to a false positive, and verified defects are kept strictly separate from optional improvements.

Works with any [`skills`](https://github.com/vercel-labs/skills)-compatible agent (Claude Code, Cursor, GitHub Copilot, Cline, and 18+ others).

## Install

```bash
npx skills add ultima95/code-validate-skill
```

Install just this skill from the repo:

```bash
npx skills add ultima95/code-validate-skill --skill code-validate
```

Preview before installing:

```bash
npx skills add ultima95/code-validate-skill --list
```

## Use

Ask your agent to validate a change, or invoke the skill directly:

```
/code-validate                 # reviews staged (else unstaged) changes
/code-validate main...HEAD     # reviews a branch/range
/code-validate src/auth.ts     # reviews specific files
```

The skill:

1. Resolves the review target (explicit paths / git range / PR → staged → unstaged → asks).
2. Sizes the diff and routes: small changes are reviewed inline; large ones are delegated per file/subsystem (or reviewed in prioritized passes).
3. Verifies every candidate issue — locate, trace execution, collect evidence, search for contradictory evidence — **before** reporting it.
4. Outputs **Verified Findings** (with severity, confidence, cited evidence, impact, and a concrete fix) separately from **Potential Improvements**.

## Claude Code companion agent (optional)

For Claude Code users, this repo also bundles a read-only subagent at
[`.claude/agents/code-validator.md`](.claude/agents/code-validator.md). The skill's large-diff path dispatches it to review each file/subsystem in an isolated context. It carries the full review policy and is granted read-only tools (`Read`, `Grep`, `Glob`, `Bash`) — it inspects and reports, never mutates code.

Claude Code auto-discovers agents under `.claude/agents/`. To use it in every project, copy it to your user agents directory:

```bash
cp .claude/agents/code-validator.md ~/.claude/agents/
```

Other agents ignore this file; the skill is fully self-contained without it.

## What's in this repo

```
skills/code-validate/SKILL.md      # the portable skill (self-contained policy + workflow)
.claude/agents/code-validator.md   # optional Claude Code companion subagent
```

## License

MIT — see [LICENSE](LICENSE).
