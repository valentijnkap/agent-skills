# create-pr-enhanced

Agent skill that opens a GitHub pull request according to the conventions and pull request template of the repository you are working in.

The skill reads the current branch, the intended base branch, and the full diff. It fills the repository template, prefers `.github/pull_request_template.md`, and also checks the usual root and docs locations. A template that exists only on the feature branch is used. The pull request is created only when you ask for it.

The skill name is `create-pr-enhanced`, so it stays separate from the built-in `create-pr` skill.

## Install

```bash
npx skills add valentijnkap/agent-skills --skill create-pr-enhanced
```

Add `-g` to install the skill for every project.

## Usage

Open a pull request with the current repository's template:

```text
/create-pr-enhanced Open a pull request for this branch into main.
```

Draft the body without opening a pull request:

```text
/create-pr-enhanced Draft the pull request body from the repository template. Do not open the pull request.
```
