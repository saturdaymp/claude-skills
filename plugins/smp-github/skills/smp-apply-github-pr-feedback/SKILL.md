---
name: smp-apply-github-pr-feedback
description: This skill should be used when the user wants to fix, address, or respond to a PR review comment on GitHub. Common triggers include "fix this PR review comment", "address this review feedback", "handle this PR comment", or when the user provides a GitHub PR review comment URL. The skill evaluates whether the feedback is valid, implements a fix if needed, commits the change, replies to the comment, and resolves the conversation.
argument-hint: <pr-review-url>
---

# Fix PR Review Comment

Process a PR review comment: evaluate validity, implement a fix if valid, commit, reply, and resolve.

## Input

The user provided this PR review URL: $ARGUMENTS

## Phase 1 - Parse Input & Fetch Review

1. Parse the GitHub URL to extract:
   - Owner and repo (from the URL path)
   - PR number (from the URL path)
   - Comment ID (from the `#discussion_r<ID>` fragment)

2. Fetch the review comment and its thread:
   ```
   gh api repos/{owner}/{repo}/pulls/comments/{comment_id}
   ```

3. Determine the root comment ID (the top-level review comment for the thread), then fetch any replies in the same thread:
   ```
   root_comment_id=$(gh api repos/{owner}/{repo}/pulls/comments/{comment_id} --jq '.in_reply_to_id // .id')
   gh api repos/{owner}/{repo}/pulls/{pr_number}/comments --jq "[.[] | select(.id == ${root_comment_id} or .in_reply_to_id == ${root_comment_id})]"
   ```

4. Display the review comment to the user, including:
   - The reviewer (who left the comment)
   - The file and line(s) referenced
   - The comment body
   - Any replies in the thread

## Phase 2 - Evaluate Validity

1. Read the code being reviewed. Use the file path and line numbers from the comment metadata (`path`, `line`, `original_line`, `diff_hunk`) to locate the relevant code.

2. Read the surrounding context in the source file to fully understand the code.

3. Assess whether the review feedback is valid:
   - Is it a real bug or correctness issue?
   - Is it a legitimate style/convention concern?
   - Is it a correct observation about the code?

4. **If NOT valid:**
   - Explain to the user why the feedback is not valid.
   - Ask the user: "Would you like me to reply to the PR comment explaining why this feedback doesn't apply, and resolve the conversation?"
   - If the user agrees:
     - Draft a polite reply explaining why the feedback is not applicable. **STOP and show the draft to the user for confirmation before posting.**
     - Post the reply using: `gh api repos/{owner}/{repo}/pulls/comments/{comment_id}/replies -f body="<reply>"`
     - Resolve the conversation (see Phase 7).
   - **STOP here. Do not proceed to Phase 3.**

5. **If valid:** Tell the user the feedback is valid and explain why, then proceed to Phase 3.

## Phase 3 - Propose Fix

1. Read the relevant source files to understand the full context around the issue.

2. Propose a specific fix. Show the user:
   - Which file(s) will be changed
   - What the change will be (describe it clearly)
   - Why this fix addresses the review feedback

3. **STOP and wait for user confirmation before implementing. Do NOT proceed until the user approves the proposed fix.**

## Phase 4 - Implement Fix

1. Apply the agreed-upon changes using the Edit or Write tools.
2. Briefly confirm what was changed.

## Phase 5 - Commit

1. Show the current changes:
   - !`git status`
   - !`git diff HEAD`

2. Draft a commit message that:
   - Describes the fix clearly
   - References the PR (e.g., "Fix review feedback on PR #123")

3. **STOP and show the draft commit message to the user. Let them confirm or provide an updated message before committing.**

4. Once confirmed, stage and commit:
   ```
   git add <specific-files>
   git commit -m "<confirmed message>"
   ```

## Phase 6 - Reply to PR

1. Get the commit SHA from the just-created commit:
   ```
   git rev-parse HEAD
   ```

2. Draft a reply to the review comment. Suggested format:
   > Fixed in <commit SHA> - <brief description of what was changed>.

3. **STOP and show the draft reply to the user. Let them confirm or edit before posting.**

4. Post the reply:
   ```
   gh api repos/{owner}/{repo}/pulls/comments/{comment_id}/replies -f body="<confirmed reply>"
   ```

## Phase 7 - Resolve Conversation

1. Get the review thread node ID using the GraphQL query in `references/graphql-queries.md`. Match the thread whose first comment has `databaseId` equal to the comment ID (or its parent if the comment is a reply).

2. Resolve the thread using the GraphQL mutation in `references/graphql-queries.md`.

3. Confirm to the user that the conversation has been resolved.

## Error Handling

- If the URL cannot be parsed (missing owner, repo, PR number, or comment ID), report the issue and ask the user to provide a valid GitHub PR review comment URL.
- If `gh api` calls fail (e.g., 404, 403), report the error clearly and ask how to proceed. Common causes: PR doesn't exist, insufficient permissions, or `gh` not authenticated.
- If the PR is already merged or closed, inform the user and ask whether to continue (the fix may still be relevant for a follow-up).
- If the review thread is already resolved, inform the user and skip Phase 7.
- Never retry failed API calls silently. Always report the error and ask the user for guidance.

## Important Notes

- At every **STOP** point, wait for the user to respond before continuing. Do not skip confirmation steps.
- Use `gh` for all GitHub API interactions, never raw HTTP requests.
- When displaying information to the user, keep it concise and focused.
