# Agent Loop

A spec-driven autonomous workflow for Claude Code. You write specs, the agent writes code and handoffs.

Inspired by [Jamon Holmgren's Night Shift workflow](https://jamon.dev/night-shift).

## How it works

```
You write specs ──→ Agent picks tasks ──→ Agent implements ──→ Agent reviews (6 personas)
   ./specs/              bugs first           + tests              loop until approved
                                                                         │
                                                                         ▼
You review handoffs ◄── Agent pushes branch ◄── Agent writes handoff
   ./handoffs/              git push             ./handoffs/<slug>.md
```

## Structure

```
├── @AGENT_LOOP.md          # The workflow — agent follows this step by step
├── @REVIEW_PERSONAS.md     # 6 review personas for the quality gate
├── specs/                  # You write these — input to the agent
│   └── _TEMPLATE.md        # Spec template
└── handoffs/               # Agent writes these — output for you
```

## Setup

1. Copy these files into your project root (or reference them with `@` includes).
2. Customize `@AGENT_LOOP.md`:
   - Add your project's validation commands in Steps 6 and 8 (look for `<!-- CUSTOMIZE -->` comments).
3. Customize `@REVIEW_PERSONAS.md`:
   - Add project-specific review concerns to each persona (e.g., encryption, specific framework patterns).
4. Write your first spec using `specs/_TEMPLATE.md`.
5. Tell Claude Code: `load @AGENT_LOOP.md and start working`.

## Key principles

- **Bugs first.** Specs tagged `type: bug` are always prioritized over features.
- **Drafts are invisible.** Specs tagged `status: draft` are skipped entirely.
- **Burn the tokens.** The 6-persona review loop runs until all approve. A human should never catch something the reviewers should have.
- **Handoffs over reports.** The agent doesn't just say what it did — it tells you exactly what to test, what to scrutinize, and what to follow up on.
- **Fix the docs, not just the code.** When something goes wrong, the root cause is usually a gap in specs, docs, or validation — not just bad code.
