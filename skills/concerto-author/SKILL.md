---
name: concerto-author
description: Author and edit Concerto model (.cto) files. Use when creating a new model, adding concepts/enums/participants/assets/transactions/events, defining relationships, applying decorators, working with namespaces and imports, or bumping a namespace version. Triggers on the presence of .cto files, on "Cannot use an unversioned import" errors, or on requests to design a business schema with Concerto.
---

# concerto-author

Help users author and edit [Concerto](https://github.com/accordproject/concerto) model files (`.cto`).

## When to use

- A new `.cto` file is being created
- Adding `concept`, `enum`, `participant`, `asset`, `transaction`, or `event` declarations
- Adding properties, relationships, or imports across namespaces
- Applying decorators (`@Term`, `@DecoratorCommand`, custom decorators)
- Versioning a namespace (`namespace org.example@1.2.3`)
- The parser throws `Cannot use an unversioned import in a versioned model`

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
4. **Always use versioned namespaces** (concerto-core@4.x enforces this). Every namespace declaration must carry a version: `namespace org.example@1.0.0`. Unversioned namespaces cause downstream parse failures.
5. **Use explicit named imports, never wildcards.** `import org.example@1.2.3.{TypeA, TypeB}` is correct; `import org.example@1.2.3.*` is not accepted in concerto@4.x.
6. **Keep `$class` in sync with the namespace version.** JSON instances must include the version suffix: `"$class": "org.example@1.0.0.MyType"`.
7. **Validate after edits** with the Concerto CLI: `npx @accordproject/concerto-cli@4 validate --ctoFiles <files> --input <data.json>`.

## Common errors

| Error | Cause | Fix |
| --- | --- | --- |
| `Cannot use an unversioned import in a versioned model` | An import path lacks `@x.y.z` | Add a version to every import |
| `Namespace is not defined for type X` | Wrong or missing version in `$class` / import | Check the exact namespace + version in the target `.cto` |
| `Wildcard imports are not supported` | `import foo.*` syntax | Replace with `import foo@x.y.z.{Type1, Type2}` |

## Reference

- [Concerto language docs](https://concerto.accordproject.org)
- [Concerto CLI](https://github.com/accordproject/concerto/tree/main/packages/concerto-cli)
- Tracking issue: [accordproject/concerto#1222](https://github.com/accordproject/concerto/issues/1222)

## Out of scope

- Generating code from a model → use `concerto-codegen`
- Validating JSON instances → use `concerto-validate`
- Migrating across Concerto major versions → use `concerto-migrate`
