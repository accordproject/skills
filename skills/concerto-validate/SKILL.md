---
name: concerto-validate
description: Validate JSON instances against a Concerto .cto model, diagnose validation errors, and generate sample conforming instances. Use when both a .cto model and a JSON instance are present, when a validation error needs explaining, or when the user wants a minimal sample instance for a model.
---

# concerto-validate

Validate JSON data against [Concerto](https://github.com/accordproject/concerto) models.

## When to use

- A `.cto` model and a JSON instance are both available
- Validation has failed and the user needs the error explained in terms of the model
- The user wants a minimal example instance conforming to a model

## Approach

1. **Identify the target type** in the model (the `$class` field of the JSON, or ask the user).
2. **Run validation** via the Concerto CLI:
   ```
   npx @accordproject/concerto-cli validate --ctoFiles <model.cto> --input <data.json>
   ```
3. **Interpret errors** by mapping the error path back to the property in the model.
4. **For sample generation:** use `concerto-cli generate` or hand-craft a minimal instance covering required properties.

## Common error patterns

- `Instance ... missing required property X` — required property absent
- `Model violation ... has no super type` — wrong/unknown `$class`
- `Expected Type X but found Y` — type mismatch, often missing `Optional` or wrong scalar

## Reference

- [Concerto CLI](https://github.com/accordproject/concerto/tree/main/packages/concerto-cli)
- Tracking issue: [accordproject/concerto#1223](https://github.com/accordproject/concerto/issues/1223)

## Out of scope

- Authoring or editing the model itself → use `concerto-author`
- Generating code from the model → use `concerto-codegen`
