# smp-apply-github-pr-feedback

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