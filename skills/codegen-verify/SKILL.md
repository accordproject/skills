---
name: codegen-verify
description: Verify that Concerto codegen output is valid in each target environment (compile, schema-check, or lint). Use when checking generated artifacts with real tools (tsc, javac, cargo, protoc, …); when adding or extending Docker/CI verification for a fromcto target; when diagnosing verification failures; or when wiring generate-and-verify into local or CI workflows.
license: Apache-2.0
metadata:
  version: 0.1.0
  author: accordproject
---

# codegen-verification

Prove that [Concerto CodeGen](https://github.com/accordproject/concerto-codegen) output is acceptable to real compilers, schema checkers, and linters not only that generation ran without crashing.

## When to use

- Confirm generated code/schemas compile or validate for a `fromcto` target
- Add or extend verification for a target (Docker image, entrypoint, Mocha suite, CI matrix)
- Debug a failing verify job or a single-target `verify:docker:<target>` / `test:verify` run
- Keep toolchain pins reproducible across local Docker and GitHub Actions

## Two layers

| Layer | Location | Role |
| --- | --- | --- |
| Local Mocha | `test/verification/` | Generate from fixtures → run host tools. Fast while developing. |
| Docker + CI | `verification/docker/` + `verify-codegen.yml` | Generate with Concerto CLI in containers → verify with real toolchains. |

Rules for both:

1. Generate from a known corpus / fixtures.
2. Hand artifacts to a **real** external tool.
3. Fail if that tool rejects them.
4. Document intentional **skips** when a pre-existing visitor bug blocks a case, never weaken the verifier to fake green.

## Supported targets

| Key | Kind | Verifier |
| --- | --- | --- |
| `typescript` | compiled | `tsc --noEmit` |
| `jsonschema` | schema | `ajv compile` |
| `graphql` | schema | `graphql` `buildSchema` |
| `protobuf` | schema | `protoc` |
| `csharp` | compiled | `dotnet build` |
| `rust` | compiled | `cargo check` |
| `java` | compiled | `javac` |
| `golang` | compiled | `go build` |
| `odata` | schema | `xmllint --noout` |
| `mermaid` | schema | `mermaid.parse` |
| `xmlschema` | schema | `xmllint --schema` |
| `openapi` | schema | `redocly lint` |
| `avro` | schema | `avro-tools idl` |
| `plantuml` | schema | `plantuml -syntax` |
| `vocabulary` | schema | VocabularyManager |
| `markdown` | lint | `markdownlint` |

Registry: `verification/docker/targets.json`. Pins: `verification/docker/versions.json`.

## Approach

1. **Pick the layer** - Mocha for day-to-day; Docker only when you need toolchain parity with CI for a **single** target.
2. **Confirm the target key** with the user (see table above; Docker key `golang` maps to CLI `GoLang`). Do not assume “all targets.”
3. **Run verification - one Docker target only:**
   ```bash
   npm run test:verify                       # local Mocha (OK when tools are on the host)
   npm run verify:docker:<target>            # e.g. verify:docker:typescript
   ```
   **Never** run `npm run verify:docker` (all targets). Full-matrix Docker is too heavy to run and confirm locally; leave the full matrix to CI.
4. **Inspect failures** - artifacts under `verification/work-<target>/` (also uploaded by CI).
5. **Extend carefully** when adding a target or corpus case:
   - Corpus / skips: `verification/corpus/manifest.json` (+ models under `verification/corpus/models/`)
   - Local cases: `test/verification/cases.js` + `*.compile.test.js` / `*.validate.test.js`
   - Docker: `verification/docker/<target>/Dockerfile` + `entrypoint.sh` (generate via `run-case.sh`, then verify)
   - Pins: bump only in `versions.json`; Dockerfiles take build-args (no hardcoded tool versions)
   - CI: matrix entry in `.github/workflows/verify-codegen.yml` (`fail-fast: false`)
   - After changes, smoke-check with **only** `npm run verify:docker:<that-target>`
6. **Keep branch codegen in the loop** - the base image symlinks this checkout into the CLI’s nested `@accordproject/concerto-codegen` so verification tests PR visitors, not a published npm copy.

## Architecture (quick map)

```text
verification/corpus/     → cases for Docker
test/verification/       → Mocha cases + suites
verification/docker/base → Node + CLI + branch codegen symlink
verification/docker/<t>  → toolchain + entrypoint (generate → verify)
scripts/verification/    → docker-run.js, docker-versions.js
.github/workflows/verify-codegen.yml
```

## Reference

- Maintainer guide: [`verification/VERIFICATION.md`](https://github.com/accordproject/concerto-codegen/blob/main/verification/VERIFICATION.md)
- Workflow: [`.github/workflows/verify-codegen.yml`](https://github.com/accordproject/concerto-codegen/blob/main/.github/workflows/verify-codegen.yml)
- Blog: [Automated Compilation Verification for Concerto Code Generation](https://accordproject.org/news/gsoc-2026-automated-compilation-verification-for-concerto-code-generation/)

## Out of scope

- Generating artifacts only (no compile/validate) → use `concerto-codegen`
- Editing the source model → use `concerto-author`
- Validating instance data against a model → use `concerto-validate`
- Running the full Docker suite locally (`npm run verify:docker`) - use one `verify:docker:<target>` and rely on CI for the matrix
- Weakening verifiers or deleting failing corpus cases to force green CI - fix the visitor or document a skip with an issue link instead
