---
name: update-pr-enhanced
description: >-
  Updates the body of the current open GitHub or Bitbucket Cloud pull request
  from its existing message and the latest commits, while preserving the
  repository's pull request template and established content. Use when the
  user asks to update, refresh, or sync an existing pull request description.
compatibility: >-
  Requires git and an existing open pull request for the current branch. For a
  GitHub remote, use a GitHub MCP server or the GitHub CLI (gh). For a
  Bitbucket Cloud remote, use the bkt CLI from avivsinai/bitbucket-cli. Do not
  use gh for Bitbucket or bkt for GitHub.
metadata:
  version: "1.0.0"
---

# update-pr-enhanced

Update the message of the existing pull request for the current repository. Start from the current PR body, then revise it using the latest committed changes and the repository's instructions. Do not create a second pull request.

This skill is separate from the built-in `update-pr` skill. Invoke it as `update-pr-enhanced`.

## Update only when asked

Update an existing pull request only when the user asks to update, refresh, sync, or rewrite its message.

When the user asks to draft, preview, or prepare the updated description, stop after showing the existing PR, title, base, head, template path, and proposed body. Do not push or edit the pull request.

## Detect the hosting platform and current pull request

Run:

```bash
git remote get-url origin
```

Use the host in that URL:

- `github.com`: GitHub. Use the GitHub MCP server or `gh`. Do not use `bkt`.
- `bitbucket.org`: Bitbucket Cloud. Use `bkt`. Do not use `gh` or a GitHub MCP server.
- Any other host: stop. This skill supports GitHub and Bitbucket Cloud only.

Accept both SSH and HTTPS remotes. Resolve the owner/workspace and repository from `origin`; do not hardcode them.

Inspect the current branch:

```bash
git branch --show-current
git status --short
```

Stop when the repository is in detached HEAD. Find the open pull request whose head/source branch is the current branch. If there is no matching open pull request, say so and stop; do not create one. If multiple open pull requests match, ask the user which one to update.

Before editing, read and retain the existing pull request's title, body, URL, base branch, and head branch. The title, base, and head must remain unchanged unless the user explicitly asks to change them.

## Inspect the latest changes

Determine the PR base from the existing pull request, then inspect the full current comparison:

```bash
git diff --stat <base>...HEAD
git diff <base>...HEAD
git log --oneline <base>..HEAD
```

Use `origin/<base>` for the comparison when that remote ref exists. Read every changed file; do not summarize from the stat or commit subjects alone. Also inspect the working tree status, but treat uncommitted and untracked files as excluded from the pull request. Mention them in the chat reply when relevant and never describe them as included changes.

If there are no commits ahead of the PR base, say so and do not rewrite the message. If the current branch is ahead, identify what is new or materially changed since the existing PR message was written. Use the latest commits as evidence, while keeping the complete base-to-head diff as the source of truth.

## Read repository instructions and template

Read instruction files that exist and apply their pull request, title, and testing rules:

- `AGENTS.md` and `.github/AGENTS.md`
- `CLAUDE.md`
- `CONTRIBUTING.md`, `.github/CONTRIBUTING.md`, and `docs/CONTRIBUTING.md`
- `.github/copilot-instructions.md`
- `.cursor/rules/` and `.cursorrules`
- A pull request section in `README.md`

Find the template in the current branch, including untracked files, using this order:

1. `.github/pull_request_template.md`
2. `.github/PULL_REQUEST_TEMPLATE.md`
3. `pull_request_template.md`
4. `PULL_REQUEST_TEMPLATE.md`
5. `docs/pull_request_template.md`
6. `docs/PULL_REQUEST_TEMPLATE.md`

When none exists, inspect markdown files in `.github/PULL_REQUEST_TEMPLATE/`, `PULL_REQUEST_TEMPLATE/`, and `docs/PULL_REQUEST_TEMPLATE/`. Use the template named by the user; when one directory has several templates and the user did not choose one, ask which applies.

The existing PR body is the starting point. Preserve its headings, order, relevant explanations, checkboxes, links, and user-provided context. Use the repository template and instructions to repair missing or stale required sections, but do not replace a meaningful existing message with a generic template or invent a ticket, plan, test, reviewer, or deployment step.

Remove obsolete claims and update sections affected by the latest commits. Describe only changes present in the base-to-head diff and only checks actually run. A test-plan item that was run is `- [x]` with the command and observed result; other items remain `- [ ]`. Do not claim tests, lint, screenshots, or review passed unless that work happened.

Keep or omit Jira and plan sections according to the existing message, repository instructions, and the same evidence rules used for new pull requests: a ticket or plan must be supplied or explicitly present in the branch, commits, or instructions. Do not invent one.

## Propose and apply the update

Before editing, summarize what will change in the PR message. Update only when the user asked for the actual update. If the user asked only for a draft, show the proposed body and leave the remote unchanged.

Preserve the existing PR title by default. Do not push, change branches, force-push, or alter git configuration. Push only when the user explicitly asks to include local commits and the remote branch is behind; otherwise updating the PR body does not require a push. Never commit uncommitted files as part of this skill.

### GitHub

Use a connected GitHub MCP server when available. Otherwise use `gh`:

```bash
gh pr view <number> --json number,url,title,body,baseRefName,headRefName,state
gh pr edit <number> --body "$(cat <<'EOF'
<updated body>
EOF
)"
```

Pass the body explicitly. Do not rely on GitHub to insert a template from the default branch. Return the pull request URL and a concise summary of the updated sections.

### Bitbucket Cloud

Use `bkt`. Check that it is installed and authenticated before editing:

```bash
bkt --version
bkt auth status
bkt context list
bkt pr list --state OPEN --limit 0 --json
```

Match `source.branch.name` to the current branch and retain the matching pull request ID, URL, and existing body. Use the matching repository context; do not pass tokens or ask the user to paste credentials. Update the body explicitly:

```bash
bkt pr edit <id> --body "$(cat <<'EOF'
<updated body>
EOF
)"
```

If `bkt` does not expose the existing body in the list output, use its documented pull-request inspection/help command to retrieve it before editing. Do not guess the old message. Return the pull request URL and a concise summary of the updated sections.

When the host command or MCP call fails, report the error and leave the pull request unchanged unless the host confirms that the edit succeeded.
