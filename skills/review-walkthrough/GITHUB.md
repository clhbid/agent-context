# GitHub recipes

Take `<owner>`, `<repo>` and `<n>` from the pull request you pinned in step 1.

## The pull request

```bash
gh pr view <n> -R <owner>/<repo> \
  --json number,title,body,author,baseRefName,headRefName,headRefOid,closingIssuesReferences,commits
gh pr diff <n> -R <owner>/<repo>
gh api user --jq .login   # who is invoking the skill
```

`closingIssuesReferences` is empty when the body links the issue some other way; read the body for
`Closes #N` and the branch name for a leading issue number.

## Checks

```bash
gh pr checks <n> -R <owner>/<repo> --json name,bucket,link
```

## Unresolved threads

REST doesn't say whether a thread is resolved; GraphQL does:

```bash
gh api graphql -F owner=<owner> -F repo=<repo> -F n=<n> -f query='
  query($owner: String!, $repo: String!, $n: Int!) {
    repository(owner: $owner, name: $repo) { pullRequest(number: $n) {
      reviewThreads(first: 100) { pageInfo { hasNextPage endCursor } nodes {
        id isResolved isOutdated path line
        comments(first: 20) { nodes { author { login } body url } } } } } } }' \
  --jq '.data.repository.pullRequest.reviewThreads.nodes[] | select(.isResolved | not)'
```

When `hasNextPage` is true, fetch the next page with
`reviewThreads(first: 100, after: "<endCursor>")`.

## Checking anchors

A comment's `line` must fall inside one of the file's hunks on the right-hand side, counting added
and context lines. Read the hunk headers (`@@ -a,b +c,d @@` covers lines `c` to `c+d-1`) from
`gh pr diff`.

## Posting the review

Write the payload as a JSON file, then post it in one request:

```json
{
  "commit_id": "<head SHA>",
  "event": "REQUEST_CHANGES",
  "body": "<summary>",
  "comments": [
    { "path": "<file>", "line": 42, "side": "RIGHT", "body": "<finding>" }
  ]
}
```

```bash
gh api repos/<owner>/<repo>/pulls/<n>/reviews --method POST --input review.json \
  --jq '.html_url + " " + .state'
```

A `422` posts nothing. `Can not request changes on your own pull request` means the reviewer is the
author; see [REVIEW.md](REVIEW.md#the-event). Read the comment count back with:

```bash
gh api repos/<owner>/<repo>/pulls/<n>/reviews/<review id>/comments --jq length
```

## Replying to and resolving a thread

```bash
gh api graphql -f id=<thread id> -f body='Addressed in <sha>.' -f query='
  mutation($id: ID!, $body: String!) {
    addPullRequestReviewThreadReply(input: { pullRequestReviewThreadId: $id, body: $body }) {
      comment { url } } }'
gh api graphql -f id=<thread id> -f query='
  mutation($id: ID!) { resolveReviewThread(input: { threadId: $id }) { thread { isResolved } } }'
```
