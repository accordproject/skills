# AGENTS.md

This file provides guidance to AI coding agents (Claude Code, Cursor, Copilot, etc.) when working with code in this repository.

## Repository Overview

A collection of skills for AI coding agents working with Vercel projects. Skills are packaged instructions and scripts that extend agent capabilities.

## Creating a New Skill

### Directory Structure

```
skills/
  {skill-name}/           # kebab-case directory name
    SKILL.md              # Required: skill definition
    scripts/              # Optional: executable scripts
      {script-name}.sh    # Bash scripts
      {script-name}.mjs   # Node scripts
    references/           # Optional: supporting docs loaded on demand
    lib/                  # Optional: shared code for scripts
```

### Naming Conventions

- **Skill directory**: `kebab-case` following `{base-tech}-{skill}` (e.g., `concerto-deploy`, `template-monitor`)
- **SKILL.md**: Always uppercase, always this exact filename
- **Scripts**: `kebab-case.sh` or `kebab-case.mjs` (e.g., `deploy.sh`, `collect-signals.mjs`)

### SKILL.md Format

```markdown
---
name: {skill-name}
description: {One sentence describing when to use this skill. Include trigger phrases like "Deploy my app", "Check logs", etc.}
license: Apache-2.0
metadata:
  version: {semantic-version-of-skill}
  author: accordproject
---

# {Skill Title}

{Brief description of what the skill does.}

## When to use

{Unnumbered list explaining the conditions to use skill}

## Approach

{Unnumbered list explaining how the skill works}

## Reference

{Unnumbered list pointing to relevant documenttation or code repositories that would be usefull to the agent}

## Out of Scope (optional)

{Unnumbered list for additional skills in this repository that is not ocvered by this skill}

## Troubleshooting

{Common issues and solutions, especially network/permissions errors}
```

### Best Practices for Context Efficiency

Skills are loaded on-demand — only the skill name and description are loaded at startup. The full `SKILL.md` loads into context only when the agent decides the skill is relevant. To minimize context usage:

- **Keep SKILL.md under 500 lines** — put detailed reference material in separate files
- **Write specific descriptions** — helps the agent know exactly when to activate the skill
- **Use progressive disclosure** — reference supporting files that get read only when needed
- **Prefer scripts over inline code** — script execution doesn't consume context (only output does)
- **File references work one level deep** — link directly from SKILL.md to supporting files

### Script Requirements

- Bash scripts: use `#!/bin/bash` and `set -e`
- Node scripts: use `#!/usr/bin/env node` and `.mjs`
- Write status messages to stderr
- Write machine-readable output (JSON) to stdout
- Include a cleanup trap for temp files when scripts create them
- Reference scripts by relative path, for example `node scripts/{script}.mjs`

### End-User Installation

Document skills.sh installation for public skills:

```bash
npx skills add vercel-labs/agent-skills --skill {skill-name}
```

For manual installs, document the native path when the skill needs one.

**Claude Code:**
```bash
cp -r skills/{skill-name} ~/.claude/skills/
```

**claude.ai:**
Add the skill to project knowledge or paste SKILL.md contents into the conversation.

If the skill requires network access, instruct users to add required domains at `claude.ai/settings/capabilities`.
