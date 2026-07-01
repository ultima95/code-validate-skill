---
name: code-validate
description: Use when asked to validate, review, or verify a code change — a diff, staged or unstaged changes, a branch, or a pull request — for real, evidence-backed defects. Triggers on "validate this diff", "review before I merge", "is this safe to ship", correctness-first low-false-positive review, or /code-validate. Not for style-only feedback or generating code.
---

# Code Validate

## Overview

Evidence-based code review. Review implementations based **only on verifiable evidence found in the repository**.

Core principle: **a missed issue is preferable to a false positive.** Never optimize for finding "more" issues — optimize for finding **real** ones. Correctness is the highest priority.

If the code does not prove something, you do not know it.

## When to use

- Validating / verifying a diff, staged changes, unstaged changes, a branch, or a PR.
- "Is this safe to ship?" — correctness-first review before merge.
- Confirming whether a specific behavior actually exists in the code.

**When NOT to use:** pure formatting/style passes, or generating new code.

## Workflow

### 1. Resolve what to review

Pick the target in this priority order:

| Input | Target |
|---|---|
| Explicit file paths given | those files |
| A git ref / range (e.g. `main...HEAD`, a commit, a tag) | that range |
| A PR number / URL | that PR's diff (`gh pr diff <n>` if available) |
| Nothing, but changes are staged | `git diff --cached` |
| Nothing, unstaged changes exist | `git diff` |
| Not a git repo, or nothing resolvable | **ask the user what to review** |

### 2. Size and route

Run `git diff --stat <target>` (or count files/lines for path lists), then:

```dot
digraph route {
    "Resolved target" [shape=box];
    "<= 5 files AND <= 400 changed lines?" [shape=diamond];
    "Isolation / thorough review requested?" [shape=diamond];
    "Review inline (this context)" [shape=box];
    "Delegate: one reviewer per file/subsystem" [shape=box];

    "Resolved target" -> "<= 5 files AND <= 400 changed lines?";
    "<= 5 files AND <= 400 changed lines?" -> "Isolation / thorough review requested?" [label="yes"];
    "<= 5 files AND <= 400 changed lines?" -> "Delegate: one reviewer per file/subsystem" [label="no"];
    "Isolation / thorough review requested?" -> "Delegate: one reviewer per file/subsystem" [label="yes"];
    "Isolation / thorough review requested?" -> "Review inline (this context)" [label="no"];
}
```

**Delegating on large reviews:** if your runtime supports subagents, dispatch one reviewer per file or subsystem so each review stays in a focused context, then aggregate. On Claude Code, use the bundled `code-validator` agent (`.claude/agents/code-validator.md`). If subagents are unavailable, review in the prioritized passes below and **explicitly state any area you did not reach.**

### 3. Verify before reporting

For every candidate issue, before you report it:

1. **Locate** the implementation. Don't stop at the first matching function — read surrounding code.
2. **Trace execution.** Follow calls, interfaces, inheritance, dependency injection, middleware, configuration, feature flags, runtime branching. Do not assume execution paths.
3. **Collect evidence:** implementation, call sites, types, schemas, tests, configuration, in-repo documentation.
4. **Search for contradictory evidence** — actively look for code that disproves your hypothesis: validation elsewhere, middleware, database constraints, type guarantees, tests covering the behavior, explicitly configured framework behavior. If found, revise your conclusion.
5. Only then report.

## Rules

- **Evidence over intuition.** No assumptions, common patterns, best guesses, framework conventions, or statistical likelihood as proof.
- **Truthfulness over completeness.** It is correct to say "Unable to verify," "Evidence not found," or "Insufficient context." Never fabricate certainty.
- **Review the implementation, not your expectations.** Distinguish incorrect / risky / unconventional / stylistic / preference. Normally only *incorrect* and *risky* block approval.
- **Never invent** runtime behavior, API contracts, missing files, architecture, deployment, infrastructure, environment variables, database schema, request lifecycle, third-party behavior, or business rules. If external knowledge is required, say so explicitly.
- **Repository is the source of truth.** If code conflicts with docs, comments, or naming, treat the implementation as authoritative unless proven otherwise.
- **No generic comments.** Never write "consider improving error handling" without where, why, evidence, and concrete consequence. If you can't make it repository-specific, omit it.
- **Separate findings from suggestions** (see output). Never present an improvement as a bug.
- **Scope control.** Review only code you inspected. If a conclusion depends on unavailable code: "Unable to complete verification because required implementation was unavailable."

## Prioritized passes (large reviews)

1. correctness
2. security
3. data integrity
4. concurrency
5. resource leaks
6. error handling
7. edge cases
8. maintainability
9. style

Never review formatting before correctness.

## Output format

### Verified Findings

For each finding:

- **Summary** — short description.
- **Severity** — Critical / High / Medium / Low.
- **Confidence** — High (directly proven by code) / Medium (strong evidence, some uncertainty) / Low (partial evidence only). Never report a Low-confidence finding as a definite bug.
- **Evidence** — File; Symbol; relevant code (with line numbers if available); execution path.
- **Reasoning** — why the evidence proves the issue.
- **Impact** — what can happen, under which conditions, who is affected. Avoid hypothetical disasters unsupported by code.
- **Suggested Fix** — concrete, consistent with the existing architecture.

### Potential Improvements

Readability, maintainability, architecture, performance, simplification. These are opinions, not defects. Keep them out of Verified Findings.

## Self-check before finalizing

- [ ] Verified every factual statement?
- [ ] Inspected the relevant implementation?
- [ ] Traced execution?
- [ ] Searched for contradictory evidence?
- [ ] Cited supporting code?
- [ ] Avoided assuming missing behavior?
- [ ] Distinguished facts from opinions?
- [ ] Would another engineer reach the same conclusion using only the repository?

If any answer is "No", revise before reporting.

## Common mistakes

- Reporting the first match without tracing execution.
- Skipping the contradictory-evidence search → false positives.
- Reviewing formatting before correctness.
- Stopping too early on a large diff instead of delegating or doing prioritized passes.
- Presenting a stylistic preference as a defect.

## Guiding principle

The repository is your only source of truth. If the repository cannot prove it, you cannot claim it. When in doubt, "I cannot verify this from the available code" beats speculation.
