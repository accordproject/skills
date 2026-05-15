---
name: template-author
description: Scaffold and edit Accord Project smart legal contract templates — the directory layout, grammar.tem.md (TemplateMark), model files, TypeScript logic, sample drafts, and package.json metadata. Use when creating a new template, adding clauses, editing template grammar/model/logic, or aligning a template with the accordproject template-library conventions.
---

# template-author

Author and edit [Accord Project](https://accordproject.org) smart legal contract templates.

## When to use

- Creating a new template from scratch
- Editing an existing template's grammar, model, or logic
- Adding a new clause to a contract-type template
- Bringing a template in line with [template-library](https://github.com/accordproject/cicero-template-library) packaging conventions

## Template anatomy

```
my-template/
  package.json          # name, version, accordproject metadata
  README.md
  text/
    grammar.tem.md      # TemplateMark grammar
    sample.md           # example instance
  model/
    model.cto           # Concerto data model
  logic/
    logic.ts            # template logic (optional)
  test/
    *.feature           # Cucumber tests (optional)
```

## Approach

1. **Decide the template kind:** `clause` (a single clause), `contract` (a full agreement). Set in `package.json` under `accordproject.template`.
2. **Author the model first** (`model/model.cto`) — define the variables the template will reference.
3. **Write the grammar** (`text/grammar.tem.md`) — TemplateMark prose with `{{variable}}` and `{{#clause}} … {{/clause}}` blocks tied to the model.
4. **Create a sample** (`text/sample.md`) — concrete instance that parses against the grammar.
5. **Verify round-trip:** parse the sample to data, draft data back to text, confirm they match.
6. **Follow template-library naming**: kebab-case, semantic versioning, populated `keywords` and `description` in `package.json`.

## Reference

- [TemplateMark](https://docs.accordproject.org/docs/markup-templatemark.html)
- [template-archive](https://github.com/accordproject/template-archive)
- [cicero-template-library](https://github.com/accordproject/cicero-template-library)
- Tracking issue: [accordproject/template-archive#912](https://github.com/accordproject/template-archive/issues/912)

## Related skills

- Authoring the model → use `concerto-author` for the `.cto` work
- Validating sample data → use `concerto-validate`
