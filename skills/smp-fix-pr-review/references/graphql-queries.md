# GraphQL Queries for PR Review Thread Resolution

## Fetch Review Thread Node IDs

Use this query to find the thread node ID for a given comment. Match the thread whose first comment has a `databaseId` equal to the target comment ID (or its parent comment ID if the target is a reply).

```graphql
query($owner: String!, $repo: String!, $pr: Int!, $after: String) {
  repository(owner: $owner, name: $repo) {
    pullRequest(number: $pr) {
      reviewThreads(first: 100, after: $after) {
        pageInfo {
          hasNextPage
          endCursor
        }
        nodes {
          id
          isResolved
          comments(first: 1) {
            nodes {
              databaseId
            }
          }
        }
      }
    }
  }
}
```

**Example invocation:**
```
gh api graphql -f query='<query above>' -f owner="owner" -f repo="repo" -F pr=123
```

> **Pagination:** If `pageInfo.hasNextPage` is `true`, re-run the query with `-f after="<endCursor>"` to fetch the next page. Repeat until the matching `databaseId` is found or all pages are exhausted.

## Resolve a Review Thread

Once the thread node ID is known, resolve it with this mutation:

```graphql
mutation($threadId: ID!) {
  resolveReviewThread(input: { threadId: $threadId }) {
    thread {
      isResolved
    }
  }
}
```

**Example invocation:**
```
gh api graphql -f query='<mutation above>' -f threadId="PRRT_abc123"
```
