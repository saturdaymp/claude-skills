# claude-skills

Custom skills for the Claude Code agent.

## Prerequisites

- [Claude Code](https://docs.anthropic.com/en/docs/claude-code) CLI installed
- [GitHub CLI (`gh`)](https://cli.github.com/) installed and authenticated

## Installation

### From GitHub

Add this repo as a plugin marketplace, then install:

```
/plugin marketplace add saturdaymp/claude-skills
/plugin install claude-skills@saturdaymp/claude-skills
```

### From a local clone

```bash
git clone https://github.com/saturdaymp/claude-skills.git
```

Then in Claude Code:

```
/plugin marketplace add /path/to/claude-skills
/plugin install claude-skills@claude-skills
```

## Skills

### smp-fix-pr-review

Fixes, addresses, or responds to a GitHub PR review comment. Given a review comment URL, it will:

1. Fetch the comment and its thread
2. Evaluate whether the feedback is valid
3. Propose and implement a fix (with your approval at each step)
4. Commit the change
5. Reply to the comment and resolve the conversation

**Usage:**

```
/smp-fix-pr-review https://github.com/owner/repo/pull/123#discussion_r1234567
```

You can also describe what you want naturally — e.g., "fix this PR review comment" and paste the URL.
