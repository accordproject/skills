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

## Using a skill

Copy the skill directory into your agent's skills location (e.g. `~/.claude/skills/<skill-name>/` for Claude Code), or reference this repo from your agent harness.

## Contributing

Issues and PRs welcome. Each skill has a tracking issue in its upstream repo (linked from the skill's `SKILL.md`).

## License

[Apache-2.0](./LICENSE)
