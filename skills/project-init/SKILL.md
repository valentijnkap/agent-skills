---
name: project-init
description: >-
  Initialize a project with a one-line README, fixed project rules in
  AGENTS.md, and a GitHub pull request template. Use when the user asks to
  initiate, start, or set up a new project, mentions project-init, or wants
  these files added to an existing project that does not have them yet.
---

# project-init

Create only these files in the project root:

- `README.md`
- `AGENTS.md`
- `.github/pull_request_template.md`

## Steps

1. Use the workspace root as the project root.
2. Take the project name from the user. If they did not give one, use the root directory name.
3. Take one sentence that describes the project. Ask for it when it was not given.
4. If `README.md`, `AGENTS.md`, or `.github/pull_request_template.md` already exists, ask whether to replace that file. Leave it unchanged when the user declines.
5. Write `README.md` from [readme.md](templates/readme.md). Replace `{{NAME}}` with the project name and `{{DESCRIPTION}}` with the sentence.
6. Write `AGENTS.md` by copying [templates/agents.md](templates/agents.md) verbatim.
7. Write `.github/pull_request_template.md` by copying [templates/pull_request_template.md](templates/pull_request_template.md) verbatim. Create `.github/` when it is missing.
8. Report which files were written and which were left unchanged.

## Templates

- README: [readme.md](templates/readme.md)
- Rules: [templates/agents.md](templates/agents.md)
- Pull request: [templates/pull_request_template.md](templates/pull_request_template.md)
