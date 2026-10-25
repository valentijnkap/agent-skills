---
name: create-pr-enhanced
description: >-
  Creates a GitHub pull request whose body follows the current repository's
  pull request template and contribution instructions. Use when the user asks
  to open, create, or draft a pull request, PR description, or PR body, or to
  follow the repository PR template.
compatibility: Requires git and GitHub access through a GitHub MCP server or the GitHub CLI (gh).
metadata:
  version: "1.0.0"
---

# create-pr-enhanced

Create a pull request for the current repository. The body follows that repository's pull request template and contribution instructions.

This skill is separate from the built-in `create-pr` skill. Invoke it as `create-pr-enhanced`.

## Create a pull request only when asked

Open or update a pull request only when the user asks to create, open, submit, or update one.

When the user asks to draft, preview, or prepare the description, stop after showing the title, base, head, template path, and body. Do not push and do not call GitHub.

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
   - Otherwise the repository default branch. Read it from `git symbolic-ref --short refs/remotes/origin/HEAD` or `gh repo view --json defaultBranchRef --jq .defaultBranchRef.name`.
   - If neither source gives a branch, ask the user. Do not assume `main`.

3. Full comparison between the intended base and the current branch:

   ```bash
   git diff --stat <base>...HEAD
   git diff <base>...HEAD
   git log --oneline <base>..HEAD
   ```

   Use `origin/<base>` for the comparison when that remote ref exists, so the diff matches what GitHub will show. The base name recorded on the pull request stays the branch name without `origin/`.

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

- Check a template checklist item only when that check was run. Leave every other item unchecked.
- State the command and the observed result for a check that was run.
- Do not claim that tests, lint, screenshots, or review passed unless that work happened.
- Do not add reviewers, assignees, labels, or deployment steps that the user did not request and the template does not ask for.

Choose the title in this order: the title the user gave, the repository's title convention, then one factual sentence from the full diff.

## Open the pull request

Do this only after the user asks for the pull request.

1. Resolve the GitHub repository from `origin`. Do not hardcode an owner or repository.
2. When an open pull request already exists for the current head, return its URL. Update its body only when the user asks for an update. Do not open a second pull request.
3. When the branch has no upstream or is ahead of its upstream, push it:

   ```bash
   git push -u origin HEAD
   ```

   Do not force-push. Do not skip hooks. Do not change git config.

4. Send the filled template explicitly as the pull request body. Do not rely on GitHub to insert a template from the default branch.

   Set the head to the current branch and the base to the intended base branch. Set draft only when the user asks for a draft.

5. Use a GitHub MCP server when one is connected. Call its create-pull-request tool with the title, body, head, and base. When the server is missing, unauthenticated, or the call fails, use the GitHub CLI:

   ```bash
   gh pr create --base "<base>" --head "<head>" --title "<title>" --body "$(cat <<'EOF'
   <filled template>
   EOF
   )"
   ```

   Add `--draft` when the user asks for a draft.

6. Return the pull request URL. When both GitHub MCP and `gh` fail, report the errors and leave the branch as it is.

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

## Safety

- Do not put secrets, tokens, credentials, or private environment values in the title, body, or commit content.
- Do not commit unless the user asks. Push the branch and open a pull request only when the user asks to create or update one.
- Do not force-push, skip hooks, or change git config.
