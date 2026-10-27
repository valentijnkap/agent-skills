# project-init

Agent skill that prepares a project with a one-line README, fixed agent rules, repository context notes, and a GitHub pull request template.

Install it from another project:

```bash
npx skills add valentijnkap/agent-skills --skill project-init
```

Add `-g` to install the skill for every project.

`SKILL.md` is the skill. When it runs, it writes these files into that project:

- `README.md` from [templates/readme.md](templates/readme.md)
- `AGENTS.md` from [templates/agents.md](templates/agents.md), with generated notes about the existing project structure and conventions
- `.github/pull_request_template.md` from [templates/pull_request_template.md](templates/pull_request_template.md)

Before writing, the skill inspects relevant existing files and records evidence-based context in `AGENTS.md`, including important directories, technologies, commands, integrations, and meaningful deviations from detected ecosystem conventions. It skips dependencies, generated output, binaries, and likely secrets. It does not treat a non-standard layout as an error by itself.

The files in `templates/` are copied or rendered into the project you initialize. They are not the rules or the documentation of this repository. If one of those files already exists, the skill asks before replacing or updating it.
