# Accord Project Agent Skills

A collection of Agent Skills for working with [Accord Project](https://accordproject.org) tooling — Concerto models and smart legal contract templates.

These skills follow the [Agent Skills](https://docs.claude.com/en/docs/claude-code/skills) format and can be loaded by any compatible agent harness (Claude Code, Claude Agent SDK, etc.).

## Skills

| Skill | Description |
| --- | --- |
| [`concerto-author`](./skills/concerto-author/) | Author and edit Concerto `.cto` model files. |
| [`concerto-validate`](./skills/concerto-validate/) | Validate JSON instances against a Concerto model and explain errors. |
| [`concerto-migrate`](./skills/concerto-migrate/) | Migrate Concerto models and data across major versions. |
| [`concerto-codegen`](./skills/concerto-codegen/) | Generate code and schema artifacts from a `.cto` model. |
| [`template-author`](./skills/template-author/) | Scaffold and edit Accord Project smart legal contract templates. |
| [`template-migrate`](./skills/template-migrate/) | Migrate Cicero templates between major versions (0.22 → 0.25, or 0.24/0.25 → 0.26). |

## Installation

```sh
npx skills add accordproject/skills
```

## Installation from Source

Both Claude Code and GitHub Copilot use the same `SKILL.md` format — just copy the skills to the right location.

**Claude Code** (user-level — available in every project):
```bash
git clone https://github.com/accordproject/skills /tmp/ap-skills && mkdir -p ~/.claude/skills && cp -r /tmp/ap-skills/skills/* ~/.claude/skills/
```

**GitHub Copilot** (project-level — add to the current repo):
```bash
git clone https://github.com/accordproject/skills /tmp/ap-skills && mkdir -p .github/skills && cp -r /tmp/ap-skills/skills/* .github/skills/
```

To install a single skill, replace the last `cp` argument with the specific directory (e.g. `…/skills/template-migrate`).

## Using a skill
Skills are automatically discovered and used when a user's request matches the skill's purpose.

For example, if a user wants to summarize a CSV file:

Locate a CSV analysis skill in the collection.

Open the AI assistant that supports skills.

Upload the CSV file.

Use a prompt such as:

Analyze this CSV file and summarize the key trends.
The assistant will identify the relevant skill, process the file, and return a summary of important insights, trends, and anomalies.

Different skills provide different capabilities, so users should review the skill documentation in the corresponding SKILL.md file for details and examples.

## Contributing

Issues and PRs welcome. Each skill has a tracking issue in its upstream repo (linked from the skill's `SKILL.md`).

## License

[Apache-2.0](./LICENSE)
