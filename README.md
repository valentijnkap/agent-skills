# agent-skills

A single repository of agent skills for coding assistants. Skills are maintained here for use with Cursor, Codex, Claude Code, and other agents that support the [skills CLI](https://github.com/vercel-labs/skills)—whichever tool you or your team uses. Each skill lives in its own directory under `skills/`.

Install every skill globally:

```bash
npx skills add valentijnkap/agent-skills --all -g
```

List available skills without installing:

```bash
npx skills add valentijnkap/agent-skills --list
```

Install one skill:

```bash
npx skills add valentijnkap/agent-skills --skill project-init
npx skills add valentijnkap/agent-skills --skill create-pr-enhanced
```

## Skills

| Skill | Description | Documentation |
| --- | --- | --- |
| `project-init` | Initialize a project with README, AGENTS.md, and a GitHub pull request template. | [skills/project-init/README.md](skills/project-init/README.md) |
| `create-pr-enhanced` | Open or draft a pull request using the current repository's PR template and contribution rules. | [skills/create-pr-enhanced/README.md](skills/create-pr-enhanced/README.md) |

## Layout

```text
agent-skills/
├── LICENSE
├── README.md
└── skills/
    ├── project-init/
    │   ├── README.md
    │   ├── SKILL.md
    │   └── templates/
    └── create-pr-enhanced/
        ├── README.md
        └── SKILL.md
```

The skills CLI discovers skills in `skills/<name>/SKILL.md`. Do not add a `SKILL.md` at the repository root; that would hide the other skills.

## Add a skill

1. Create `skills/<name>/` with `SKILL.md` and `README.md`.
2. Set the `name` field in frontmatter to match the directory name.
3. Add a row to the table in this README.
4. Verify discovery: `npx skills add . --list` from this repository.

## License

[MIT](LICENSE). You may use, copy, and change these skills, including the templates that `project-init` writes into a project.
