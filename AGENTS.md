# AGENTS.md

This is a Claude Code plugin that provides skills for PR review workflows. It follows the [Claude Code plugin structure](https://docs.anthropic.com/en/docs/claude-code/plugins).

## Testing

No automated tests. Verify skill changes manually by running the skill against a real PR review comment URL.

## Key Constraint

Skills in this plugin use a phase-based workflow with explicit STOP points where the agent must wait for user confirmation. Never remove these gates — they are intentional for user safety.
