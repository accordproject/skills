---
name: concerto-author
description: Author and edit Concerto model (.cto) files. Use when creating a new model, adding concepts/enums/participants/assets/transactions/events, defining relationships, applying decorators, working with namespaces and imports, or bumping a namespace version. Triggers on the presence of .cto files or requests to design a business schema with Concerto.
---

# concerto-author

Help users author and edit [Concerto](https://github.com/accordproject/concerto) model files (`.cto`).

## When to use

- A new `.cto` file is being created
- Adding `concept`, `enum`, `participant`, `asset`, `transaction`, or `event` declarations
- Adding properties, relationships, or imports across namespaces
- Applying decorators (`@Term`, `@DecoratorCommand`, custom decorators)
- Versioning a namespace (`namespace org.example@1.2.3`)

## Approach

1. **Read the existing model first.** Understand the namespace, version, imports, and existing types before adding anything.
2. **Choose the right declaration kind:**
   - `concept` — pure data structure, no identity
   - `asset` — uniquely identifiable thing (`identified by`)
   - `participant` — actor in the system
   - `transaction` — submittable action, time-stamped
   - `event` — emitted notification
   - `enum` — closed set of values
3. **Use scalars and `Optional` properties** rather than nullable workarounds. Properties default to required.
4. **Use versioned namespace imports** (`import org.example@1.2.3.{TypeA, TypeB}`) when the source namespace is versioned.
5. **Validate after edits** with the Concerto CLI: `npx @accordproject/concerto-cli parse --ctoFiles <files>`.

## Reference

- [Concerto language docs](https://concerto.accordproject.org)
- [Concerto CLI](https://github.com/accordproject/concerto/tree/main/packages/concerto-cli)
- Tracking issue: [accordproject/concerto#1222](https://github.com/accordproject/concerto/issues/1222)

## Out of scope

- Generating code from a model → use `concerto-codegen`
- Validating JSON instances → use `concerto-validate`
- Migrating across Concerto major versions → use `concerto-migrate`
