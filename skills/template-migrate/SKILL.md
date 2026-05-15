---
name: template-migrate
description: Migrate Accord Project smart legal contract templates between Cicero major versions. Use when bumping `@accordproject/cicero-core` (e.g. 0.22/0.24/0.25 → 0.26), when `Template.fromDirectory` throws because of an unversioned namespace or a removed API (`Clause.parse`, `template.getParserManager()`), when the `concerto` CLI fails to install transitively, or when `@accordproject/template-engine@3.x` rejects a previously-working `grammar.tem.md` for missing optional guards.
---

# template-migrate

Migrate Cicero templates and template-library repositories across Cicero major versions. Cicero 0.25/0.26 changed enough surface area that a working 0.24 template will typically refuse to load, build, or render until each of the items below is addressed.

## When to use

- A template's CI fails with `The template targets Cicero version ^X but the current Cicero version is Y`.
- `Template.fromDirectory` throws `Cannot use an unversioned import …` or `Namespace is not defined for type …`.
- `Clause.parse`, `template.getParserManager()`, `toArchive('ergo')`, or `concerto-tools.FileWriter` references in your build pipeline throw at runtime.
- `template-engine` rejects an existing grammar with `Optional properties used without guards: …`.
- Workspace `compile` scripts fail with `sh: 1: concerto: not found` or, after install, a `SyntaxError: Unexpected token ':'` from `@stoplight/spectral-core`.

## Approach

Work in this order — earlier steps unblock the later ones.

1. **Pin the toolchain at the new versions.** In the repo root `package.json`:
   - `@accordproject/cicero-core` → 0.26.x
   - `@accordproject/concerto-core` → 4.x (matches cicero 0.26)
   - `@accordproject/markdown-cicero` / `markdown-html` → 0.18.x
   - `@accordproject/concerto-cli` → 4.x (provides the `concerto` binary used by workspace `compile` scripts)
   - Add an `overrides.ajv: "8.17.1"` entry. `concerto-cli@4` transitively pulls `@stoplight/spectral-core`, which resolves `ajv@^8.6.2` and currently picks the broken `8.20.0` — the override clamps it back to a working release. Track [accordproject/concerto#1221](https://github.com/accordproject/concerto/pull/1221) and drop the override once it lands.

2. **Bump every workspace's `accordproject.cicero` range** in `src/*/package.json` to `^0.26.0` (or the new target). The cicero-core constructor refuses to load templates whose range doesn't satisfy the installed version.

3. **Version every Concerto namespace** referenced from the template's `model/*.cto`:
   - The local namespace declaration: `namespace org.accordproject.foo` → `namespace org.accordproject.foo@0.1.0`.
   - Every import: `import org.accordproject.signature.ContractSigned from …signature@0.2.0.cto` → `import org.accordproject.signature@0.3.0.ContractSigned from …signature@0.3.0.cto`. The remote `.cto` file at the URL must itself declare a versioned namespace — check by `curl`ing the URL and looking for `namespace foo@x.y.z`. If only an unversioned version exists upstream, find the next published model that ships with a version (e.g. `signature@0.3.0`, `connect@0.4.0`).
   - Wildcard imports (`import foo.* from …`) are no longer accepted — replace with explicit `{Type1,Type2}` lists.
   - Update every `$class` literal in `request.json`, `sample.json`, `logic.ts`, and `logic.test.ts` to include the new version (e.g. `com.docusign.connect.X` → `com.docusign.connect@0.4.0.X`).

4. **Re-author `grammar.tem.md` for `template-engine@3.x` strictness.** Bare references to optional properties throw at draft time. Wrap each optional with `{{#optional fieldName}}{{this}}{{/optional}}`. The error message lists the offending property names.

5. **Refresh the build script (`run.js`-style code).** Cicero 0.26 removed several APIs the legacy `cicero-template-library` `run.js` used:
   - `new Clause(template); instance.parse(sampleText)` — no replacement. Either commit a `sample.json` per template (bootstrap from older archives if needed — see *Bootstrap data*) or synthesise via `template.getFactory().newResource(ns, name, id, { generate: true, includeOptionalFields: true })` then `template.getSerializer().toJSON(...)`.
   - `template.getParserManager().getTemplate()` — gone. Read `text/grammar.tem.md` from disk directly.
   - `template.toArchive('ergo')` — gone. Use `'typescript'` (or `'es6'`).
   - `require('@accordproject/concerto-tools').FileWriter` — moved to `@accordproject/concerto-util`.
   - Wrap `factory.newResource(...)` calls in try/catch: relationships pointing at abstract types (e.g. `--> Contract` from `signature@0.3.0.ContractSigned`) throw "No concrete extending type for …"; treat as a non-fatal warning.

6. **Sanity-check the template type.** `Template.fromDirectory` now validates that `accordproject.template` in `package.json` matches the `@template` asset's parent: `extends Clause` → `"template": "clause"`; `extends Contract` → `"template": "contract"`. Mismatches throw "No concrete extending type for …".

7. **Run the rendering test** against `@accordproject/template-engine`'s `TemplateArchiveProcessor.draft(data, 'markdown', {})` — this catches grammar/model mismatches the older toolchain swallowed. Several known engine bugs surface at this stage; mark them with `it.fails(...)` rather than `it.skip(...)` so vitest turns red when an upstream fix lands. Open issues at the time of writing:
   - `#ulist`/`#olist` with direct variable refs throws "Paths must be supplied" — [accordproject/template-engine#145](https://github.com/accordproject/template-engine/issues/145)
   - Inline `{{% expr %}}` Formula nodes missing `value` — [accordproject/template-engine#146](https://github.com/accordproject/template-engine/issues/146)
   - `draft()` can't see TS helpers from `logic/logic.ts` — [accordproject/template-engine#147](https://github.com/accordproject/template-engine/issues/147)
   - `markdown-template` calls removed `property.isPrimitive()` on concerto 4 — [accordproject/markdown-transform#673](https://github.com/accordproject/markdown-transform/issues/673)

## Bootstrap data

If existing templates lose their natural-language→JSON parser (every cicero ≥ 0.26 template does), regenerate `sample.json` per template by:

1. `npm install @accordproject/cicero-core@0.22.2` in a side directory.
2. For each template name, download `https://templates.accordproject.org/archives/<name>@<latest>.cta`.
3. Use the old cicero-core `Template.fromArchive` + `new Clause(template).parse(samples.default)` + `instance.getData()` to produce JSON.
4. Commit the resulting JSON as `sample.json` in the new template directory. Note the `$class` will use the *old* unversioned namespace — the render path that consumes it should detect a mismatch with `template.getTemplateModel().getFullyQualifiedName()` and fall back to a synthesised instance.

## Build hygiene

While migrating, also flip these conventions — they make subsequent builds an order of magnitude faster and remove flake from network model fetches:

- Remove `@*accordproject*.cto` and `src/*/logic/generated` from `.gitignore` and commit both. The `@models...cto` files are the local cache cicero-core writes when it first resolves an external import; committing them makes builds offline and deterministic. The `logic/generated/` directory holds the `concerto compile` output that workspace logic imports at runtime (enums like `TemporalUnit` are real values, not type-only).
- Remove any `pretest`/`prebuild` hook that runs `npm run compile --workspaces` — it spawns one Node process per workspace and dominates wallclock once the generated files are committed.
- For large libraries, run the per-template work in `run.js` under a bounded `p-limit` pool (concurrency 8 is plenty); cicero is I/O-bound and parallelises trivially once external models are cached on disk.

## Reference

- [Cicero](https://github.com/accordproject/cicero)
- [template-engine](https://github.com/accordproject/template-engine)
- [cicero-template-library](https://github.com/accordproject/cicero-template-library) — reference migration in [PR #484](https://github.com/accordproject/cicero-template-library/pull/484)

## Related skills

- Updating the underlying `.cto` files → use `concerto-migrate`
- Authoring a new template from scratch → use `template-author`
- Validating sample/request data against the new model → use `concerto-validate`
