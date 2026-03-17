# Agent Loop

This document defines the autonomous workflow for completing tasks. Follow these steps sequentially. Do not skip steps.

## Step 0: Orient

- Read `CLAUDE.md` and/or `AGENTS.md` in whichever repo(s) this task touches.
- Read `@REVIEW_PERSONAS.md` so you understand the review criteria you'll be held to later.
- Identify which repo(s) and areas of the codebase this task involves.

## Step 1: Pick a Task

Look in `./specs/` for spec files. Pick a task using this priority order:

1. **Bugs** — any spec tagged `type: bug`. Always work bugs first.
2. **Features / enhancements** — any spec tagged `type: feature` or `type: enhancement`.

**Skip rules:**
- Ignore any spec tagged `status: draft`. These are not ready.
- If a spec has `depends_on: other-spec` and that dependency hasn't been completed yet, skip it — work the dependency first.
- If no specs are available, stop and report that there is nothing to work on.

Once you've picked a task, announce which spec you're working on and why (priority reasoning).

### Stacked specs

If a spec has `depends_on`, it forms a stack. The agent loop handles stacks as follows:

1. Work the base spec (the one with no `depends_on`) first, following Steps 2–9 as normal.
2. After the review loop passes for the base spec, **do NOT push yet**. Stay on the branch.
3. Pick up the dependent spec next. Run Steps 2–9 again for the new spec, committing on top of the existing work.
4. After the review loop passes for the stacked spec, push the entire branch (Step 11). The branch now contains both tasks as a clean commit history.
5. The handoff (Step 12) should cover both specs.

This keeps related work on one branch with a linear commit history. If the stack is deeper than 2, keep chaining — only push after the final spec in the stack passes review.

## Step 2: Understand the Spec

Read the full spec carefully. Before writing any code:

- Summarize the problem/feature in your own words.
- List the acceptance criteria explicitly.
- Identify edge cases and potential risks.
- Note which repo(s) and areas of the codebase are affected.
- If anything is ambiguous, stop and ask rather than assuming.

## Step 3: Load Context

Based on the spec, read the relevant source files you'll be modifying or that inform the change. This includes:

- Directly affected files (components, controllers, services, endpoints, etc.)
- Related test files to understand existing coverage.
- Any docs referenced in the spec or the repo's `CLAUDE.md`.

Do not start implementation until you have a solid mental model of the affected code.

## Step 4: Plan

Write a brief implementation plan covering:

- What files will be created or modified.
- The approach and why it's the right one.
- What tests need to be written or updated.
- Any migrations, schema changes, or cross-repo coordination needed.
- Potential risks or things you're unsure about.

Present the plan. In supervised mode, wait for human approval before proceeding. If running autonomously, proceed after a self-check that the plan is sound.

## Step 5: Create Branch

Create a new branch from the latest `main` (or default branch) in the affected repo(s).

### Branch naming

If the spec has a `tracking:` field linking to an issue tracker, try to fetch the branch name from the issue (e.g., via MCP tool or CLI). This keeps branches consistent with tracker-generated names.

<!-- CUSTOMIZE: Add your issue tracker integration here, e.g.:
- Linear: `mcp__linear-server__get_issue` returns branch name in metadata
- GitHub Issues: use `gh issue view` to get context for naming
- Jira: use Jira MCP or CLI to fetch branch name
-->

If the tracker is unavailable or the spec has no `tracking:` field, fall back to: `agent/<spec-slug>` (e.g., `agent/fix-sync-race-condition`).

For stacked specs, name the branch after the base spec — the stack builds on one branch.

```bash
git checkout main && git pull && git checkout -b <branch-name>
```

If the task spans multiple repos, create a branch in each with the same name.

## Step 6: Implement

Write the code. Follow the conventions in each repo's `CLAUDE.md`. Key principles:

- **Simplest thing that works.** Don't over-engineer.
- **Tests alongside code.** Write or update tests as you go, not as an afterthought.
- **Small, logical commits.** Commit after each meaningful unit of work with a clear message describing what and why.
- **Don't break the build.** Run the relevant checks frequently as you work.

<!-- CUSTOMIZE: Add your project's quick validation commands here, e.g.:
### Quick validation during implementation
- `npm test` / `yarn test`
- `npm run lint` / `yarn lint`
- `npm run typecheck` / `yarn typecheck`
-->

## Step 7: Self-Review

Before involving review personas, do your own review:

- Re-read every changed file diff (`git diff`).
- Check for: leftover debug code, commented-out code, missing error handling at system boundaries, hardcoded values that should be configurable, security issues (injection, XSS, auth bypass).
- Verify all acceptance criteria from the spec are met.
- Ensure tests cover the meaningful behavior, not just happy paths.

Fix any issues found before proceeding.

## Step 8: Run Full Checks

Run the complete validation suite for the affected repo(s).

<!-- CUSTOMIZE: Add your project's full check commands here, e.g.:
- `npm run allchecks`
- `yarn typecheck && yarn lint && yarn test`
-->

All checks must pass. If anything fails, fix it and re-run. Do not proceed with failures.

## Step 9: Review Loop

This is the most important quality gate. Spawn subagents to review your changes through the lens of each persona defined in `@REVIEW_PERSONAS.md`.

### Process

1. For each persona, spawn a subagent with:
   - The full diff of changes (`git diff main...HEAD`)
   - The original spec
   - The persona's review instructions from `@REVIEW_PERSONAS.md`
   - Access to read any file in the repo for context

2. Each reviewer returns one of:
   - **APPROVED** — no issues found from their perspective.
   - **CHANGES REQUESTED** — with specific, actionable feedback.

3. If ANY reviewer requests changes:
   - Address all feedback.
   - Re-run full checks (Step 8).
   - Re-run the review loop from the top (all personas review again).

4. Loop until all 6 personas return APPROVED in the same round.

**Do not skip this loop.** Do not push with outstanding review feedback. Burn the tokens. The human reviewing this branch should not find issues that these reviewers should have caught.

## Step 10: Final Commit

Ensure all changes are committed with clear, descriptive messages. The final commit message on the branch should summarize:

- What was done and why (reference the spec).
- Any notable decisions or trade-offs made.

## Step 11: Push Branch

Push the branch to the remote. Do NOT create a PR.

```bash
git push -u origin agent/<spec-slug>
```

If the task spans multiple repos, push all branches.

## Step 12: Handoff

Write a handoff file to `./handoffs/<spec-slug>.md` for the human reviewer. This is YOUR deliverable to THEM — make it actionable, not just informational.

Use this structure:

```markdown
# Handoff: <spec title>

**Branch:** `agent/<spec-slug>`
**Spec:** `specs/<spec-filename>`
**Repo(s):** ...

## What Changed

<!-- 2-5 bullet summary. What did you do and why. Keep it brief — they'll read the diff. -->

## Manual Test Plan

<!-- Step-by-step instructions for the human to verify this works. Be specific:
     - What to click / what URL to visit
     - What to look for (expected behavior)
     - Edge cases to try (empty state, dark mode, mobile, etc.)
     Think: if you could only test 5 things by hand, what would they be? -->

- [ ] ...

## Decisions & Trade-offs

<!-- Things you chose between alternatives. For each one:
     - What you decided
     - What the alternative was
     - Why you went this way
     The human may disagree — that's the point. Flag these so they can. -->

## Review With Extra Scrutiny

<!-- Specific files, functions, or patterns where you're least confident.
     Maybe you weren't sure about the right pattern, or the CSS might
     not match pixel-perfect, or you made an assumption about an edge case.
     Point the human exactly where to look. -->

- [ ] `path/to/file.ts:L42` — reason to look closely

## Cleanup & Follow-ups

<!-- Things you noticed but intentionally didn't fix (out of scope),
     plus any unrelated issues you bumped into. Each should be actionable
     enough to become its own spec if the human agrees. -->

- [ ] ...
```

The handoff file is the contract between the agent and the human. A good handoff means the human spends their time on judgment calls, not on figuring out what happened.

---

## Notes

- **When things go wrong:** If the agent produces incorrect code, don't just fix the code. Analyze WHY the wrong thing was produced. Is the spec ambiguous? Are the docs wrong? Is there a missing convention? Fix the root cause.
- **Docs are living:** If you discover that project docs are wrong, incomplete, or misleading during your work, note it in the report. Better docs make future loops better.
- **One task per loop.** Complete one spec fully before starting another. Quality over throughput. The exception is stacked specs — these are worked sequentially on one branch by design.
