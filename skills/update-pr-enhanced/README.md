# update-pr-enhanced

Agent skill that refreshes the body of the existing open GitHub or Bitbucket Cloud pull request from its current message and the latest committed changes. It preserves meaningful existing content, follows the repository's pull request template, and does not create a second pull request.

The skill name is `update-pr-enhanced`, so it stays separate from the built-in `update-pr` skill.

## Install

```bash
npx skills add valentijnkap/agent-skills --skill update-pr-enhanced
```

Add `-g` to install the skill for every project.

## Usage

Update the current pull request description:

```text
/update-pr-enhanced Update the pull request message with the latest commits.
```

Draft the updated body without editing the pull request:

```text
/update-pr-enhanced Draft an updated pull request body. Do not edit the pull request.
```
