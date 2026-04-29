---
name: aiden-check-ci
version: 1.0.0
description: Check CI/CD status for a branch or PR using Aiden's GitHub MCP tools
---

## MANDATORY: Use Aiden MCP Tools

The Aiden MCP server provides GitHub tools that are ALREADY in your tool list.
They work exactly like Read, Bash, Edit — you call them as tool invocations.
They are named with the prefix `mcp__aiden__github_`.

**Authentication is handled automatically by the MCP server.** You do NOT need
GitHub tokens, gh CLI auth, SSH keys, .netrc files, environment variables, or
any credentials. The tools work immediately with zero setup.

### NEVER do any of the following for GitHub API access:
- `curl` or `wget` to api.github.com
- `gh` CLI commands (gh pr, gh api, etc.)
- `env | grep` or scanning for tokens/secrets
- `cat ~/.netrc`, `git credential`, or `ssh -T git@github.com`
- Any attempt to find, construct, or configure GitHub authentication
- Installing packages or CLIs for GitHub access

If an MCP tool call fails, report the error to the user. Do NOT fall back to
CLI alternatives.

### Verify tools are available

Before starting, confirm you can see `mcp__aiden__github_*` tools in your
available tools. If they are NOT available, STOP and tell the user:
"The GitHub MCP tools are not available. Please check the sandbox MCP configuration."

### Tool parameters

All GitHub PR tools require these parameters:
- `owner` (string): GitHub org or username, e.g. "supatest-ai"
- `repo` (string): Repository name, e.g. "aiden"
- `prNumber` (integer): PR number, e.g. 42

Example tool call:
```
Tool: mcp__aiden__github_get_pull_request
Parameters: { "owner": "supatest-ai", "repo": "aiden", "prNumber": 42 }
```

### Resolving owner/repo

If the user only provides a PR number, run `git remote get-url origin` to get
the remote URL, then parse owner and repo from it. This is the ONLY git CLI
command you should run for GitHub operations. Everything else uses MCP tools.

### Available GitHub MCP tools

| Tool name | Purpose |
|-----------|---------|
| `mcp__aiden__github_get_pull_request` | Get PR details (title, state, labels, merge status) |
| `mcp__aiden__github_get_pr_diff` | Get unified diff of a PR |
| `mcp__aiden__github_list_pr_comments` | List all comments on a PR |
| `mcp__aiden__github_list_pr_reviews` | List all reviews on a PR |
| `mcp__aiden__github_list_pr_files` | List changed files with additions/deletions |
| `mcp__aiden__github_get_issue` | Get issue details |
| `mcp__aiden__github_get_ci_status` | Get CI check run status for a ref |
| `mcp__aiden__github_add_comment` | Add a comment to an issue or PR |
| `mcp__aiden__github_create_pr_review` | Submit a review (APPROVE/REQUEST_CHANGES/COMMENT) |
| `mcp__aiden__github_add_labels` | Add labels to an issue or PR |
| `mcp__aiden__github_merge_pull_request` | Merge a PR |
| `mcp__aiden__github_close_issue` | Close an issue or PR |
| `mcp__aiden__github_request_reviewers` | Request reviewers on a PR |

---

## Task: Check CI/CD Status

### Input

The user will provide one of:
- A PR number or URL
- A branch name
- A commit SHA
- Nothing (use current branch)

### Workflow

1. **Resolve the ref**:
   - If PR number given: call `mcp__aiden__github_get_pull_request` to get the head SHA
   - If branch name given: use it directly as the ref
   - If nothing given: run `git branch --show-current` to get the current branch

2. **Resolve owner/repo**: if not provided, run `git remote get-url origin` and parse.

3. **Check CI status**: call `mcp__aiden__github_get_ci_status` with
   `{ owner, repo, ref }` where ref is the branch name, tag, or SHA.

4. **Report**:
   - Overall status: Passing | Failing | Pending
   - Individual check runs with name and status
   - If a check failed, include the html_url link for full logs
     (the MCP tool does not return log output)

5. **PR merge readiness** (if checking a PR): also report on:
   - Review status (call `mcp__aiden__github_list_pr_reviews`)
   - PR state and merge status (from the get_pull_request call in step 1)
