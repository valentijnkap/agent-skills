---
name: project-init
description: >-
  Initialize a project with a one-line README, project rules and repository
  context in AGENTS.md, and a GitHub pull request template. Use when the user
  asks to initiate, start, or set up a new project, mentions project-init, or
  wants these files added to an existing project.
---

# project-init

Create only these files in the project root:

- `README.md`
- `AGENTS.md`
- `.github/pull_request_template.md`

Do not modify source code or other existing files. The project context is
written into `AGENTS.md`; it is not a separate generated file.

## Steps

1. Use the workspace root as the project root.
2. Take the project name from the user. If they did not give one, use the root directory name.
3. Take one sentence that describes the project. Ask for it when it was not given.
4. Inspect the existing project before generating files. Start with the directory tree and then read relevant documentation, manifests, build and test configuration, CI configuration, and obvious entry points. Skip `.git`, dependency/vendor directories, build output, generated files, binaries, and files that may contain secrets. Do not make changes during inspection.
5. Record only evidence-based context for the agent. Use these headings when there is relevant information: `Observed project`, `Technology and tooling`, `Directory layout`, `Development and test commands`, `Integrations and constraints`, and `Potential convention deviations`. Include source paths or filenames for non-obvious claims. Distinguish observed facts from inferences and do not invent missing information. Omit empty sections.
6. Compare the observed directory layout with the conventions that apply to the detected ecosystem. Record only meaningful or potentially surprising deviations, with the path and evidence that caused the observation. A different layout is not automatically a problem.
7. If `README.md`, `AGENTS.md`, or `.github/pull_request_template.md` already exists, ask whether to replace or update that file. Leave it unchanged when the user declines. If `AGENTS.md` is left unchanged, do not write generated project context elsewhere.
8. Write `README.md` from [readme.md](templates/readme.md). Replace `{{NAME}}` with the project name and `{{DESCRIPTION}}` with the sentence.
9. Write `AGENTS.md` from [agents.md](templates/agents.md). Replace `{{PROJECT_CONTEXT}}` with the generated context notes. Keep the rules in the template and keep the context concise.
10. Write `.github/pull_request_template.md` by copying [templates/pull_request_template.md](templates/pull_request_template.md) verbatim. Create `.github/` when it is missing.
11. Report which files were written, updated, or left unchanged, and mention when context could not be added because `AGENTS.md` was left unchanged.

## Templates

- README: [readme.md](templates/readme.md)
- Rules and context: [templates/agents.md](templates/agents.md)
- Pull request: [templates/pull_request_template.md](templates/pull_request_template.md)
