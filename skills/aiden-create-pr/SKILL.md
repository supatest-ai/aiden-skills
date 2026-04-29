---
name: aiden-create-pr
version: 1.0.0
description: Create a GitHub pull request with post-creation MCP operations
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

## Task: Create a Pull Request

This task uses BOTH local `git` commands (for VCS operations) and MCP tools
(for GitHub API operations). The boundary is clear:

- **`git` (local VCS)**: status, branch, push, log, diff — local operations only
- **MCP tools**: everything that talks to the GitHub API

Note: There is no MCP tool for creating a PR. Use `git` to push the branch,
then use the Bash tool to run: `gh pr create --title "..." --body "..."`
This is the ONLY acceptable use of the `gh` CLI.

### Input

The user may provide:
- A title and/or description for the PR
- A target base branch (defaults to main/master)
- Or just ask you to "create a PR" and you infer from context

### Workflow

1. **Check branch state** — use `git` to determine:
   - Current branch name (`git branch --show-current`)
   - Uncommitted changes (`git status --short`)
   - Commits ahead of base (`git log main..HEAD --oneline`)
   - Remote tracking status (`git status -sb`)

2. **Analyze changes** — use `git` to understand the diff:
   - `git log main..HEAD --oneline` for commit history
   - `git diff main...HEAD --stat` for changed files summary
   - Read the diff to understand what changed

3. **Draft PR title and body**:
   - Title: under 70 characters, descriptive
   - Body: summary bullets, test plan, breaking changes if any

4. **Push and create PR**:
   - Push: `git push -u origin HEAD`
   - Create: `gh pr create --title "..." --body "..."`

5. **Post-creation** (if requested by user) — use MCP tools:
   - Add labels: call `mcp__aiden__github_add_labels` with
     `{ owner, repo, issueNumber: prNumber, labels: ["label1"] }`
   - Request reviewers: call `mcp__aiden__github_request_reviewers` with
     `{ owner, repo, prNumber, reviewers: ["username"] }`
   - Check CI: call `mcp__aiden__github_get_ci_status` with
     `{ owner, repo, ref: "branch-name" }`

### Output

Report the PR URL and a summary of what was created.
