---
name: concerto-migrate
description: Migrate Concerto models and JSON data across major Concerto versions. Use when upgrading a project's Concerto dependency, when a model uses deprecated syntax, or when JSON data fails validation after a model change. Covers namespace versioning, decorator changes, and import syntax shifts between Concerto 1.x, 2.x, and 3.x.
---

# concerto-migrate

Migrate [Concerto](https://github.com/accordproject/concerto) models and data between major versions.

## When to use

- Upgrading `@accordproject/concerto-core` across major versions
- A `.cto` file uses syntax that no longer parses
- JSON instances fail validation after a model upgrade

## Approach

1. **Detect the current version** from `package.json` and from CTO syntax markers (presence/absence of versioned namespaces, decorator syntax, etc.).
2. **Identify the target version** the user wants to reach.
3. **Apply migration rules** in order:
   - 0.82 → 1.0: namespace and type system overhaul
   - 1.0 → 2.0: scalar types, decorator command set
   - 2.0 → 3.0: versioned namespaces become first-class, import syntax
4. **Migrate data** where shapes changed — most JSON data carries over unchanged but `$class` values may need namespace-version suffixes.
5. **Run `concerto-validate`** after migration to confirm both model and data are consistent.

## Reference

- [Concerto migration guides in techdocs](https://docs.accordproject.org/docs/ref-migrate-concerto-2.0-3.0.html)
- Tracking issue: [accordproject/concerto#1224](https://github.com/accordproject/concerto/issues/1224)

## Out of scope

- Net-new model authoring → use `concerto-author`
