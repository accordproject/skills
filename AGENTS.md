# AGENTS.md

Guidance for AI coding agents and contributors working in this repository.

## Repository overview

This repository contains Agent Skills for Accord Project tools, including Concerto models and smart legal contract templates. Each skill is documented in a `SKILL.md` file under `skills/`.

## Repository structure

- `skills/<skill-name>/SKILL.md` — instructions for one Accord Project skill.
- `README.md` — skill catalog, installation guidance, and contribution links.
- `skills.sh.json` — categories used to organize skills in the skills.sh registry.

## Editing a skill

- Keep the front matter consistent with the skill directory name and the other skills in this repository. Include a clear `description` with useful activation cues, the license, and the existing metadata fields.
- Keep each skill focused on its stated task. Link to a related skill when work belongs to that skill's scope.
- Ground commands, API names, and migration steps in the relevant Accord Project documentation or source. State the supported tool or package versions when behavior depends on a version, and recheck version-sensitive guidance before changing it.
- Make examples complete enough to adapt, and ensure commands use the documented CLI and flags.

## Adding or renaming a skill

- Add the skill under `skills/<skill-name>/SKILL.md` using a kebab-case directory name that matches the front matter `name`.
- Update the skill catalog in `README.md` and the relevant grouping in `skills.sh.json` so the new skill is discoverable.
- Check links and names after changing a skill directory or its references.
