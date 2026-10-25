---
name: create-pr-enhanced
description: >-
  Creates a pull request on GitHub or Bitbucket Cloud whose body follows the
  current repository's pull request template and contribution instructions.
  Use when the user asks to open, create, or draft a pull request, PR
  description, or PR body, or to follow the repository PR template.
compatibility: >-
  Requires git. For a GitHub remote, use a GitHub MCP server or the GitHub
  CLI (gh). For a Bitbucket Cloud remote, use the bkt CLI from
  avivsinai/bitbucket-cli. Do not use gh for Bitbucket or bkt for GitHub.
metadata:
  version: "1.1.0"
---

# create-pr-enhanced

Create a pull request for the current repository. The body follows that repository's pull request template and contribution instructions.

This skill is separate from the built-in `create-pr` skill. Invoke it as `create-pr-enhanced`.

## Create a pull request only when asked

Open or update a pull request only when the user asks to create, open, submit, or update one.

When the user asks to draft, preview, or prepare the description, stop after showing the title, base, head, template path, and body. Do not push and do not call GitHub or Bitbucket.

## Detect the hosting platform

Run:

```bash
git remote get-url origin
```

Use the host in that URL:

- `github.com`: GitHub. Use the GitHub MCP server or `gh`. Do not use `bkt`.
- `bitbucket.org`: Bitbucket Cloud. Use `bkt`. Do not use `gh` or a GitHub MCP server.
- Any other host: stop. This skill supports GitHub and Bitbucket Cloud only.

Accept both SSH and HTTPS remotes. Strip a trailing `.git` and any userinfo from an HTTPS URL. Read the owner and repository from the path. On Bitbucket Cloud the first path segment is the workspace and the second is the repository slug. Do not hardcode an owner, workspace, or repository.

## Gather the branches and the full diff

Inspect the repository before writing the description. Run the independent git commands together.

1. Current branch, which is the pull request head:

   ```bash
   git branch --show-current
   git status --short
   git diff
   git diff --staged
   ```

   Stop when the repository is in detached HEAD. There is no branch to use as head.

2. Intended base branch, in this order:
   - The base the user named.
   - Otherwise the repository default branch. Read it from `git symbolic-ref --short refs/remotes/origin/HEAD`.
   - On GitHub, when that ref is missing, read `gh repo view --json defaultBranchRef --jq .defaultBranchRef.name`.
   - On Bitbucket Cloud, when that ref is missing, ask the user. Do not call `gh`.
   - If neither source gives a branch, ask the user. Do not assume `main`.

3. Full comparison between the intended base and the current branch:

   ```bash
   git diff --stat <base>...HEAD
   git diff <base>...HEAD
   git log --oneline <base>..HEAD
   ```

   Use `origin/<base>` for the comparison when that remote ref exists, so the diff matches what the host will show. The base name recorded on the pull request stays the branch name without `origin/`.

   Read every changed file. When the patch is large, read it per file until each path in the stat has been reviewed. Do not describe the pull request from the stat line alone.

4. Committed changes form the pull request. Uncommitted and untracked files are not part of it. Mention them in the chat reply and do not describe them as included changes. Do not commit them unless the user asks.

   When the current branch has no commits ahead of the base, say so and stop. Do not open an empty pull request.

## Read the repository instructions

Read the instruction files that exist and apply their pull request, title, and testing rules:

- `AGENTS.md` and `.github/AGENTS.md`
- `CLAUDE.md`
- `CONTRIBUTING.md`, `.github/CONTRIBUTING.md`, and `docs/CONTRIBUTING.md`
- `.github/copilot-instructions.md`
- `.cursor/rules/` and `.cursorrules`
- A pull request section in `README.md`

The template controls which sections appear and in which order. Use the instructions to fill those sections and to choose the title style.

## Find the pull request template

Search the working tree of the current branch, including untracked files. A template that exists only on the feature branch is valid. Do not skip it because the base branch does not have it.

Use the first file that exists:

1. `.github/pull_request_template.md`
2. `.github/PULL_REQUEST_TEMPLATE.md`
3. `pull_request_template.md`
4. `PULL_REQUEST_TEMPLATE.md`
5. `docs/pull_request_template.md`
6. `docs/PULL_REQUEST_TEMPLATE.md`

When none of those files exist, look for markdown files in:

- `.github/PULL_REQUEST_TEMPLATE/`
- `PULL_REQUEST_TEMPLATE/`
- `docs/PULL_REQUEST_TEMPLATE/`

Use the template the user names. When the user does not name one and a directory contains a single template, use that file. When several templates remain, ask which one to use.

When the repository has no template, say so and use this fallback, in this order:

```markdown
## Summary

## Test plan
- [ ] Step to verify the change
```

## Fill the template

Write the body by copying the template's sections and order, then replacing its prompts with facts from the diff, commits, instructions, and checks.

- Replace example text, sample issue keys, `TODO` lines, and placeholders such as `[description]`, `<ticket>`, and `{summary}`.
- Remove HTML comments that only tell the author what to write, after following those comments.
- Keep the template's headings, labels, and required lists.
- When the template says to delete a section that does not apply, delete it.
- Leave every other section in place. For a section with nothing to report, write a short factual statement such as "None."

Omit a Jira section when no ticket exists. Omit a Plan section when no plan exists. This includes a heading or labeled field named `Jira`, `Jira ticket`, `Jira issue`, `Plan`, `Implementation plan`, or `Technical plan`.

A ticket counts only when the user supplied one, or when an explicit key such as `ABC-123` appears in the branch name, commit messages, or repository instructions. A plan counts only when the user supplied one or pointed to a plan document. Do not invent a ticket, a link, or a plan. When one exists, keep its section and fill it.

Describe only changes that are in the base-to-head diff and only checks that were actually run.

Write every test-plan item as a markdown checkbox. A check that was run is `- [x]` and includes the command and the observed result. Every other test point stays `- [ ]`. Do this even when the template lists those steps as plain bullets or prose. Do not add test points the template does not ask for, except in the fallback template above.
- Do not claim that tests, lint, screenshots, or review passed unless that work happened.
- Do not add reviewers, assignees, labels, or deployment steps that the user did not request and the template does not ask for.

Choose the title in this order: the title the user gave, the repository's title convention, then one factual sentence from the full diff.

## Open the pull request

Do this only after the user asks for the pull request.

1. Resolve the repository from `origin`. Do not hardcode an owner, workspace, or repository.
2. Send the filled template explicitly as the pull request body. Do not rely on the host to insert a template from the default branch.

   Set the head to the current branch and the base to the intended base branch. Set draft only when the user asks for a draft.

3. Follow the GitHub or Bitbucket section below. When an open pull request already exists for the current head, return its URL. Update its body only when the user asks for an update. Do not open a second pull request.

   Push only when a pull request is about to be created, or when the user asked to update an existing one and the local branch is ahead. Skip the push when the remote branch already matches the current branch:

   ```bash
   git push -u origin HEAD
   ```

   Do not force-push. Do not skip hooks. Do not change git config. On Bitbucket Cloud, run the `bkt` checks below before this push.

### GitHub

Use a GitHub MCP server when one is connected. Call its create-pull-request tool with the title, body, head, and base. When the server is missing, unauthenticated, or the call fails, use the GitHub CLI:

```bash
gh pr create --base "<base>" --head "<head>" --title "<title>" --body "$(cat <<'EOF'
<filled template>
EOF
)"
```

Add `--draft` when the user asks for a draft.

Look for an existing open pull request for the current head before creating one. Then push with the command above. Return the pull request URL. When both GitHub MCP and `gh` fail, report the errors and leave the branch as it is.

### Bitbucket Cloud

1. Check that `bkt` is installed:

   ```bash
   bkt --version
   ```

   When the command is missing, tell the user how to install it and stop. Do not install it unless the user explicitly agrees. On macOS with Homebrew:

   ```bash
   brew install avivsinai/tap/bitbucket-cli
   ```

   For other install methods, point to the [CLI repository](https://github.com/avivsinai/bitbucket-cli).

2. Check authentication and contexts:

   ```bash
   bkt auth status
   bkt context list
   ```

   Do not show or ask for token values. `bkt` stores credentials in the OS keychain. Do not pass `--token` or `--with-token`, and do not ask the user to paste a token.

   When no host is authenticated, stop and tell the user to sign in in the browser:

   ```bash
   bkt auth login https://bitbucket.org --kind cloud --web
   ```

3. Use a context whose workspace and repository match `origin`. An active context is marked with `*` in `bkt context list`. When a matching context exists but is not active, activate it:

   ```bash
   bkt context use <name>
   ```

   When the CLI is authenticated and no context matches `origin`, derive the workspace and repository from `origin` and create a local context. Name it after the repository slug:

   ```bash
   bkt context create <name> --host api.bitbucket.org --workspace <workspace> --repo <repository> --set-active
   ```

4. List open pull requests before creating one. `bkt pr list --json` returns an object with a `pull_requests` array, not a bare array. `--limit 0` returns every open pull request:

   ```bash
   bkt pr list --state OPEN --limit 0 --json
   ```

   Match `source.branch.name` to the current branch. When one matches, return its `links.html.href`. Update that pull request only when the user asks for an update:

   ```bash
   bkt pr edit <id> --body "$(cat <<'EOF'
   <filled template>
   EOF
   )"
   ```

5. After the checks above, push with the command above when the branch has no upstream or is ahead of its upstream. Then create the pull request with the title, source branch, target branch, and filled template:

   ```bash
   bkt pr create --title "<title>" --source "<head>" --target "<base>" --body "$(cat <<'EOF'
   <filled template>
   EOF
   )"
   ```

   Add `--draft` when the user asks for a draft. Do not add reviewers unless the user asks.

6. Return the pull request URL from the command output. When `bkt` fails, report the error and leave the branch as it is.

## Example

Template found at `.github/pull_request_template.md`:

```markdown
## Summary
<!-- Replace with one or two sentences. -->
[Describe the change]

## Jira
<!-- Example: PROJ-123 -->

## Plan
[Link the implementation plan]

## Test plan
- [ ] Unit tests
- [ ] Manual check
```

The branch `feat/export` is one commit ahead of `main`. `npm test` passed. The user did not provide a ticket or a plan and asked to open the pull request.

Body to send:

```markdown
## Summary
Add CSV export for the account activity report.

## Test plan
- [x] Unit tests (`npm test` passed)
- [ ] Manual check
```

The Jira and Plan sections are omitted. The manual check stays unchecked because it was not run. The head is `feat/export` and the base is `main`.

On GitHub, pass that body to `gh pr create --base main --head feat/export --title "Add CSV export for the account activity report."`.

On Bitbucket Cloud, pass the same body to:

```bash
bkt pr create --title "Add CSV export for the account activity report." --source feat/export --target main --body "$(cat <<'EOF'
## Summary
Add CSV export for the account activity report.

## Test plan
- [x] Unit tests (`npm test` passed)
- [ ] Manual check
EOF
)"
```

Do not add `--draft` here. The user asked to open the pull request.

## Git in a sandbox

When a Git command fails inside a sandbox with an `xcode-select` or Command Line Tools message, and Git works in the user's Terminal, rerun that same Git command with the sandbox escalation the agent is allowed to use. Do this before recommending a Command Line Tools install or a macOS update.

## Safety

- Do not put secrets, tokens, credentials, or private environment values in the title, body, commit content, or chat reply.
- Do not commit unless the user asks. Push the branch and open a pull request only when the user asks to create or update one.
- Do not force-push, skip hooks, or change git config.
- Do not install `bkt` unless the user explicitly agrees.
