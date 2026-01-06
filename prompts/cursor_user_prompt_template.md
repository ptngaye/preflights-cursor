# Cursor User Prompt Templates

Use these templates to start implementation after Preflights has completed clarification.

## Option 1: Use Generated Agent Prompt (Recommended)

After Preflights completes, paste the content of `docs/AGENT_PROMPT.md` directly into Cursor chat.

```
[Paste content of docs/AGENT_PROMPT.md here]
```

The agent prompt already includes:
- Task reference
- Architecture context
- Implementation constraints
- Acceptance criteria

## Option 2: Minimal Prompt

If you don't want to open files, use this minimal prompt:

```
Read docs/CURRENT_TASK.md and docs/ARCHITECTURE_STATE.md, then implement the task.
Follow the allowlist strictly and satisfy all acceptance criteria.
```

## Option 3: Quick Reference Prompt

For quick reference during implementation:

```
Continue implementing the task defined in docs/CURRENT_TASK.md.
- Only modify files in the Allowlist section
- Never touch files in Forbidden section
- Ensure all Acceptance Criteria are met
```

## Starting a New Feature

When you want to implement a new feature:

```
I want to [describe feature].
Please check if docs/CURRENT_TASK.md exists. If not, use the require_clarification tool to start the process.
```

## Resuming Work

If you need to resume work on an existing task:

```
Please read docs/CURRENT_TASK.md and continue the implementation.
Show me what's left to do based on the Acceptance Criteria.
```

## Architecture Questions

Before making technical decisions:

```
Before implementing, please call read_architecture to understand the current state.
Then help me decide on [technical question].
```

## Tips

1. **Let Preflights generate the task** - Don't manually create CURRENT_TASK.md
2. **Trust the allowlist** - It's there to prevent scope creep
3. **Check acceptance criteria** - They define "done"
4. **Reference ADRs** - Past decisions have context you might need
