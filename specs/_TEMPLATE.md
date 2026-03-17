---
type: bug | feature | enhancement
status: draft | ready
repo: # which repo(s) this touches
priority: high | medium | low
tracking: # optional — link to issue tracker (Linear, GitHub Issues, Jira, etc.)
depends_on: other-spec-filename  # optional — stack this task on top of another
---

# Title

<!-- One line. What is the change? -->

## Problem

<!-- What's broken, missing, or inadequate? Be specific.
     For bugs: what happens vs. what should happen. Include reproduction steps.
     For features: what user need or business goal does this serve? -->

## Solution

<!-- How should this be solved? You don't need to specify exact code, but be
     clear enough that someone unfamiliar with your thinking could implement it
     without guessing your intent.

     If there are multiple viable approaches, state which one you chose and why. -->

## Affected Areas

<!-- Which parts of the codebase does this touch? Think through the full path:
     - Components / UI
     - State management / stores
     - Services / business logic
     - Data access / repositories
     - API endpoints
     - Workers / background jobs
     - Migrations -->

## Acceptance Criteria

<!-- A checklist of concrete, verifiable outcomes. The agent will use these to
     know when the work is done. Be precise — vague criteria lead to vague work.

     - [ ] When X happens, Y should occur
     - [ ] The Z endpoint returns the correct shape
     - [ ] Existing behavior for A is unchanged -->

- [ ] ...

## Edge Cases

<!-- Things that are easy to miss. Think about:
     - Empty / null / missing data
     - Unauthorized or unauthenticated access
     - Concurrent operations / race conditions
     - Migration from existing data
     - Different user roles or permission levels -->

## Out of Scope

<!-- Explicitly state what this spec does NOT cover. This prevents scope creep
     and helps the agent stay focused. If something is related but should be a
     separate spec, say so. -->

## References

<!-- Links to issues, Slack threads, documentation, related PRs, or
     anything else that provides context. -->
