---
name: template-migrate
description: Migrate Accord Project smart legal contract templates between Cicero major versions. Use when bumping `@accordproject/cicero-core` (e.g. 0.22 → 0.25 or 0.22/0.24/0.25 → 0.26), when a template has `.ergo` logic files that need converting to TypeScript, when Cucumber `.feature` tests need replacing with Jest/Vitest, when `Template.fromDirectory` throws because of an unversioned namespace or a removed API (`Clause.parse`, `template.getParserManager()`), when the `concerto` CLI fails to install transitively, or when `@accordproject/template-engine@3.x` rejects a previously-working `grammar.tem.md` for missing optional guards.
license: Apache-2.0
metadata:
  version: 0.1.0
  author: accordproject
---

# template-migrate

Migrate Cicero templates and template-library repositories across Cicero major versions.

---

## When to use

- A template has `.ergo` logic files and/or Cucumber `.feature` tests — it is on **0.22** and needs migrating to **0.25** (TypeScript runtime).
- A template's CI fails with `The template targets Cicero version ^X but the current Cicero version is Y`.
- `Template.fromDirectory` throws `Cannot use an unversioned import …` or `Namespace is not defined for type …`.
- `Clause.parse`, `template.getParserManager()`, `toArchive('ergo')`, or `concerto-tools.FileWriter` references in your build pipeline throw at runtime.
- `template-engine` rejects an existing grammar with `Optional properties used without guards: …`.
- Workspace `compile` scripts fail with `sh: 1: concerto: not found` or, after install, a `SyntaxError: Unexpected token ':'` from `@stoplight/spectral-core`.

---

## Migration path overview

| Source | Target | Key change |
|--------|--------|-----------|
| 0.22 (Ergo) | 0.25 (TypeScript) | Replace `.ergo` logic with `logic.ts`, update namespace convention, replace Cucumber with Jest/Vitest |
| 0.24/0.25 | 0.26 | Bump toolchain, version Concerto namespaces, patch removed APIs |

---

## Part 1 — Migrating from 0.22 (Ergo) to 0.25 (TypeScript)

### Template categories

Identify the category before starting — it determines which steps are required.

| Category | Description | Examples |
|----------|-------------|---------|
| **A** — Logic | Has `logic/` with `.ergo` files | `helloworld`, `latedeliveryandpenalty`, `volumediscount` |
| **B** — Data-only | No `logic/`, no tests | `company-information`, `signature-block-title-name-date` |
| **C** — Stateful | Has `state.json` and state-using logic | `helloworldstate`, `perishable-goods` |

Category B only needs Steps 1, 2, 6, 7, and 8. Skip Steps 3, 4, and 5.

### Overview of changes

| Area | 0.22 (before) | 0.25 (after) |
|------|--------------|-------------|
| Logic language | Ergo (`.ergo`) | TypeScript (`logic.ts`) |
| Logic class (stateless) | `contract X over Y { clause ... }` | `class X extends TemplateLogic<ITemplateModel>` |
| Logic class (stateful) | `contract X over Y { clause ... }` | `class X extends TemplateLogic<ITemplateModel, IStateType>` |
| Model namespace | `org.accordproject.<name>` (unversioned) | `io.clause.<name>@<semver>` (versioned) |
| Main asset name | Custom (e.g. `HelloWorldClause`) | Always `TemplateModel` |
| Main asset base | `Clause` or `Contract` | Always `Clause` |
| `@template` decorator | Not used | Required on `TemplateModel` |
| State definition | `state.json` file | `concept MyState identified { ... }` in model |
| Events | `emit Obligation{...}` in Ergo | `event MyEvent { ... }` in model, returned in `events: object[]` |
| Model imports | Wildcard, remote URL, unversioned | Selective, remote URL, versioned |
| Local model copies | Not present | Required in `model/` directory |
| Generated TS types | Not applicable | `logic/generated/` from `concerto compile` |
| Cicero version | `^0.22.0` | `^0.25.0` |
| Runtime field | `ergo` (or absent) | `typescript` |
| Test framework | Cucumber/Gherkin `.feature` files | Jest/Vitest (`logic.test.ts`) with mocked runtime globals |

### Step 1 — Update `package.json`

```json
// Before
{
    "accordproject": { "template": "clause", "cicero": "^0.22.0" },
    "devDependencies": { "cucumber": "^5.1.0" },
    "scripts": { "test": "cucumber-js test -r .cucumber.js" }
}

// After
{
    "accordproject": { "runtime": "typescript", "template": "clause", "cicero": "^0.25.0" }
}
```

- Set `accordproject.cicero` to `"^0.25.0"`
- Add `accordproject.runtime: "typescript"`
- Remove `devDependencies` (cucumber) and `scripts.test` blocks
- Bump `version` (patch or minor)

### Step 2 — Update the model (`model/model.cto`)

#### 2a. Version the namespace

```
// Before
namespace org.accordproject.helloworld

// After
namespace io.clause.helloworld@0.1.0
```

Start all migrations at `@0.1.0`.

#### 2b. Update imports to versioned, selective form

```
// Before (wildcard, unversioned)
import org.accordproject.contract.* from https://models.accordproject.org/accordproject/contract.cto
import org.accordproject.runtime.* from https://models.accordproject.org/accordproject/runtime.cto

// After (selective, versioned)
import org.accordproject.contract@0.2.0.Clause from https://models.accordproject.org/accordproject/contract@0.2.0.cto
import org.accordproject.runtime@0.2.0.{Request,Response} from https://models.accordproject.org/accordproject/runtime@0.2.0.cto
```

Common versioned import mappings:

| Old import | New import |
|-----------|-----------|
| `org.accordproject.contract.*` from `.../contract.cto` | `org.accordproject.contract@0.2.0.Clause` from `.../contract@0.2.0.cto` |
| `org.accordproject.runtime.*` from `.../runtime.cto` | `org.accordproject.runtime@0.2.0.{Request,Response}` from `.../runtime@0.2.0.cto` |
| `org.accordproject.time.*` from `.../time@0.2.0.cto` | `org.accordproject.time@0.3.0.{Duration,TemporalUnit}` from `.../time@0.3.0.cto` |
| `org.accordproject.party.*` | Remove — Party references are not supported |
| `org.accordproject.obligation.*` | Remove — replace with `event` declarations |
| `org.accordproject.money.*` | Remove — replace with plain `Double` fields |

Only import the specific types you actually use (no wildcards).

#### 2c. Rename main asset to `TemplateModel` and add `@template`

```
// Before
asset HelloWorldClause extends Clause {
  o String name
}

// After
@template
asset TemplateModel extends Clause {
  o String name
}
```

- Must be named `TemplateModel`, must extend `Clause`, must have `@template`
- If the old asset extended `Contract`, change it to `Clause`
- Remove `--> Party` relationship fields

#### 2d. Add `event` declaration (if template emits events — Categories A & C)

```
event LateDeliveryAndPenaltyEvent {
  o Boolean penaltyCalculated
}
```

#### 2e. Add state `concept` (Category C — stateful templates only)

Replace `state.json` with a `concept ... identified` in the model:

```
concept LateDeliveryAndPenaltyState identified {
  o Integer count
  o Boolean lateDeliveryProcessed default=false
  o Double totalPenalties default=0.0
}
```

#### 2f. Copy dependent model files locally

Download and place in `model/` with naming convention `@<host>.<path-with-dots>@<version>.cto`:

```bash
curl -o model/@models.accordproject.org.accordproject.contract@0.2.0.cto \
  https://models.accordproject.org/accordproject/contract@0.2.0.cto
curl -o model/@models.accordproject.org.accordproject.runtime@0.2.0.cto \
  https://models.accordproject.org/accordproject/runtime@0.2.0.cto
curl -o model/@models.accordproject.org.time@0.3.0.cto \
  https://models.accordproject.org/time@0.3.0.cto
```

### Step 3 — Generate TypeScript types from the model (Categories A & C)

```bash
npx @accordproject/concerto compile \
  --model model/model.cto \
  --target typescript \
  --output logic/generated
```

Install globally to use Cocnerto CLI directly: `npm install -g @accordproject/concerto-cli`

Re-run whenever `model/model.cto` changes.

### Step 4 — Replace Ergo logic with TypeScript (Categories A & C)

Delete all `.ergo` files from `logic/`. Create `logic/logic.ts`.

Three types are **injected by the runtime** — use suppression comments, do not import them:

| Runtime type | Suppression |
|-------------|------------|
| `TemplateLogic<TModel, TState?>` | `// @ts-ignore` on the `class` line |
| `EngineResponse<TState>` | `// @ts-expect-error` on the interface line |
| `InitResponse<TState>` | `// @ts-expect-error` on return annotation |

#### Stateless template example (helloworld)

```typescript
import { IMyRequest, IMyResponse, ITemplateModel } from "./generated/io.clause.helloworld@0.1.0";

type HelloWorldResponse = { result: IMyResponse; }

// @ts-ignore
class HelloWorldLogic extends TemplateLogic<ITemplateModel> {
    async trigger(data: ITemplateModel, request: IMyRequest): Promise<HelloWorldResponse> {
        return {
            result: {
                output: `Hello ${data.name} ${request.input}`,
                $timestamp: new Date(),
                $class: 'io.clause.helloworld@0.1.0.MyResponse'
            }
        };
    }
}
export default HelloWorldLogic;
```

#### Stateful template additions (Category C)

- Add state type as second type parameter: `TemplateLogic<ITemplateModel, IMyState>`
- `trigger` takes a third parameter `state: IMyState`
- Add `async init(data: ITemplateModel): Promise<InitResponse<IMyState>>` returning initial state
- Trigger response must include `state` and `events` alongside `result`

#### Ergo → TypeScript translation table

| Ergo pattern | TypeScript equivalent |
|-------------|----------------------|
| `contract.fieldName` | `data.fieldName` |
| `return ResponseType{ ... }` | `return { result: { ..., $timestamp: new Date(), $class: '...' }, state, events }` |
| `enforce condition else throw Error{...}` | `if (!condition) throw new Error('...')` |
| `isBefore(date1, date2)` | `date1 < date2` |
| `now()` | `new Date()` |
| `diffDurationAs(d, unit)` | Manual ms arithmetic helper |
| `emit PaymentObligation{...}` | Declare `event` in model; return in `events: [event]` |
| String concatenation `++` | Template literals or `+` |
| `min([a, b])` | `Math.min(a, b)` |

Types from `org.accordproject.time` (e.g. `TemporalUnit`) may not be available at runtime — redeclare as inline enums/interfaces in `logic.ts` rather than importing from `./generated/`.

### Step 5 — Write unit tests (`logic/logic.test.ts`)

Mock runtime globals **before** importing logic:

```typescript
// @ts-nocheck
declare global { var TemplateLogic: any; var EngineResponse: any; var InitResponse: any; }

(global as any).TemplateLogic = class TemplateLogic<T, S> {
    async init(data: T): Promise<any> { return { state: {} }; }
    async trigger(data: T, request: any, state: S): Promise<any> { return {}; }
};
(global as any).EngineResponse = class EngineResponse<S> {};
(global as any).InitResponse = class InitResponse<S> {};

import MyLogic from './logic';
import { ITemplateModel, IMyRequest } from './generated/io.clause.mytemplate@0.1.0';

describe('MyLogic', () => {
    it('should compute the expected result', async () => {
        const logic = new MyLogic();
        const result = await logic.trigger(model, request, state);
        expect(result.result).toHaveProperty('$class');
        expect(result.result).toHaveProperty('$timestamp');
    });
});
```

For stateful templates: test `init` separately, test state accumulation by passing `result.state` as `state` to the next call.

### Step 6 — Update request/response data files

Update all `$class` values to versioned namespaces:

```json
// Before
{ "$class": "org.accordproject.latedeliveryandpenalty.LateDeliveryAndPenaltyRequest" }

// After
{ "$class": "io.clause.latedeliveryandpenalty@0.1.0.LateDeliveryAndPenaltyRequest" }
```

Also update nested `$class` values (e.g. `org.accordproject.time.Duration` → `org.accordproject.time@0.3.0.Duration`).

### Step 7 — Update `text/grammar.tem.md`

Template grammar syntax (`{{variable}}`, `{{#if ...}}`, `{{/if}}`) is unchanged. However, replace Party relationship variables with plain string fields if any are used.

### Step 8 — Remove legacy files

Delete from each template:
- `logic/*.ergo` — all Ergo files
- `test/*.feature` — Cucumber scenarios
- `.cucumber.js` — test runner setup
- `state.json` — state is now in the model `concept`

### Step 9 — Verify the migration

```javascript
const { TemplateArchiveProcessor } = require('@accordproject/template-engine');
const { Template } = require('@accordproject/cicero-core');

const template = await Template.fromDirectory('./src/helloworld');
const processor = new TemplateArchiveProcessor(template);
const data = { "$class": "io.clause.helloworld@0.1.0.TemplateModel", "name": "World",
               "clauseId": "test-id", "$identifier": "test-id" };
const draft = await processor.draft(data, 'markdown', {});
const response = await processor.trigger(data, { input: "everyone" });
```

### Per-template checklist

- [ ] `package.json` — `runtime: typescript`, `cicero: ^0.25.0`, cucumber removed
- [ ] `model/model.cto` — namespace `io.clause.<name>@<ver>`, imports versioned/selective
- [ ] `model/model.cto` — main asset is `TemplateModel` with `@template`, extends `Clause`
- [ ] `model/model.cto` — state `concept MyState identified { ... }` (Category C)
- [ ] `model/model.cto` — `event MyEvent { ... }` (if applicable)
- [ ] `model/` — local copies of all imported `.cto` files
- [ ] `logic/generated/` — TypeScript interfaces from `concerto compile`
- [ ] `logic/logic.ts` — implements `TemplateLogic<ITemplateModel[, IState]>`, default export
- [ ] `logic/logic.test.ts` — Jest/Vitest tests with mocked runtime globals
- [ ] `logic/*.ergo` — deleted
- [ ] `test/*.feature` and `.cucumber.js` — deleted
- [ ] `state.json` — deleted
- [ ] `request.json` — `$class` updated to versioned namespace

### Special cases

**Templates extending `Contract`:** Change to `Clause`, rename asset to `TemplateModel`.

**Party relationships (`--> Party buyer`):** Not supported. Replace with plain `String` fields (`o String buyerName`) and update grammar accordingly.

**`org.accordproject.obligation.*` imports:** Replace `emits PaymentObligation` with an `event` declaration in the model; return instances in the `events` array from `trigger`.

**`org.accordproject.money.MonetaryAmount` fields:** Replace with `Double` (and optionally `String` for currency code) or a custom concept.

**Multiple `.ergo` files (e.g. `hellomodule`):** Consolidate all logic into a single `logic/logic.ts`.

---

## Part 2 — Migrating from 0.25/0.26

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
- [cicero-template-library MIGRATION.md](https://github.com/accordproject/cicero-template-library/blob/main/MIGRATION.md) — 0.22 → 0.25 reference
- [cicero-template-library PR #484](https://github.com/accordproject/cicero-template-library/pull/484) — reference migration

## Related skills

- Updating the underlying `.cto` files → use `concerto-migrate`
- Authoring a new template from scratch → use `template-author`
- Validating sample/request data against the new model → use `concerto-validate`
