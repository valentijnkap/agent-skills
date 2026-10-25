# project-init

Agent skill that prepares a project with a one-line README, fixed agent rules, and a GitHub pull request template.

Install it from another project:

```bash
npx skills add valentijnkap/agent-skills --skill project-init
```

Add `-g` to install the skill for every project.

`SKILL.md` is the skill. When it runs, it writes these files into that project:

- `README.md` from [templates/readme.md](templates/readme.md)
- `AGENTS.md` from [templates/agents.md](templates/agents.md)
- `.github/pull_request_template.md` from [templates/pull_request_template.md](templates/pull_request_template.md)

The files in `templates/` are copied into the project you initialize. They are not the rules or the documentation of this repository. If one of those files already exists, the skill asks before replacing it.
