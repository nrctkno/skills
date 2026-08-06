---
name: peer-review
description: Review a GitHub pull request for correctness/bugs, style/convention adherence, and security issues, and assess whether existing discussion comments on the PR raise valid concerns. Use when asked to review a PR, do a code review, review these changes, or check a pull request before merge. Prints findings to chat only — does not post comments to GitHub.
compatibility: Requires gh CLI (authenticated)
---

Review the given GitHub pull request and report findings in chat. Never post comments, reviews, or approvals to GitHub — this skill is read-only with respect to GitHub state.

## Steps

1. **Load the PR.**
   - `gh pr view <number> --json title,body,baseRefName,headRefName,files,comments,reviews` for context and existing discussion comments.
   - `gh api repos/{owner}/{repo}/pulls/{number}/comments` for inline review comments (each tied to a specific file/line).
   - `gh pr diff <number>` for the actual diff. Read full changed files (not just the diff hunks) when the surrounding code is needed to judge correctness.

2. **Review across exactly three lenses.** Skip a lens entirely if the diff has nothing relevant to it (e.g. a docs-only PR has nothing for "security").
   - **Correctness & bugs**: logic errors, off-by-one/edge cases, incorrect assumptions, unhandled error paths, race conditions, behavior that contradicts the PR's stated intent.
   - **Style & conventions**: naming, structure, and patterns inconsistent with the rest of the touched files or the broader repo. Don't flag pure taste — only real inconsistency with existing conventions.
   - **Security**: injection, auth/authz gaps, secret handling, unsafe input handling, unsafe deserialization, SSRF/path traversal, and similar OWASP-style issues.

3. **Classify every finding into exactly one severity:**
   - `blocking` — must be fixed before merge (bug, security issue, broken behavior).
   - `suggestion` — worth doing, not merge-blocking (style drift, missing edge-case handling that's low-risk, better naming).
   - `nit` — trivial, purely optional polish.

4. **Verify before reporting.** Don't flag something from a partial read of a diff hunk alone — check the surrounding file when in doubt. A plausible-looking bug that turns out to be already handled elsewhere is not a finding.

5. **Evaluate existing discussion comments independently.** For every comment already on the PR (issue-style and inline review comments), judge — from the diff itself, not from the comment's tone or confidence — whether it points to a real problem in the current code. A comment can be `valid` (the concern holds up against the actual diff), `not valid` (the concern doesn't hold — already handled, based on a misreading, or resolved by a later commit), or `unclear` (can't be confirmed without more context, e.g. a design opinion with no objectively right answer).

## Output format

First, the severity-tiered findings from your own review, grouped by severity (blocking first), each with a `file:line` reference and a one-sentence explanation of the concrete failure scenario — not just what looks off, but what breaks and under what input/condition:

```
## Blocking
- path/to/file.ts:42 — <what breaks, and when>

## Suggestions
- path/to/file.ts:88 — <what's inconsistent or missing>

## Nits
- path/to/file.ts:12 — <trivial polish>
```

Then, in a separate section, your assessment of each existing discussion comment:

```
## Existing Comments
- @author on path/to/file.ts:23 — valid: <why the concern holds up>
- @author on path/to/file.ts:57 — not valid: <why it doesn't hold up>
- @author (general) — unclear: <why it can't be confirmed either way>
```

Omit any section with nothing to report (including "Existing Comments" if the PR has no comments). If there are no findings at all, say so plainly — don't invent nits to fill space.
