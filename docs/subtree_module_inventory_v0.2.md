# Subtree Module Inventory v0.2

This document records the current step-by-step repository analysis for third-party candidates.

The focus is not generic reuse. The focus is identifying the smallest *real working unit* suitable for one of the following outcomes:

- `whole_repo subtree`
- `split_subdir subtree`
- `package dependency`
- `reference_only`
- `reject`

## Decision criteria

A repository is a good subtree candidate when it has:

- a clear package or module boundary;
- a stable runtime entrypoint;
- a bounded build/test surface;
- a good reason to vendor and patch locally.

A repository remains `reference_only` when it is primarily useful for:

- schema design;
- processor ordering;
- pipeline architecture;
- option surface design;
- semantic export patterns.

## Status legend

- `reviewed`: package/build/test/entrypoint boundary has been inspected
- `adopt_subtree`: preferred subtree candidate
- `package_dependency`: better consumed via upstream package manager
- `reference_only`: keep as design or architecture reference
- `defer`: boundary known, but import postponed

---

## Reviewed repositories

| id | repository | category | status | preferred import mode | working boundary |
|---|---|---|---|---|---|
| `pixelmatch` | `mapbox/pixelmatch` | `fallback_validation` | reviewed | `whole_repo subtree` or package dependency | root package |
| `looks_same` | `gemini-testing/looks-same` | `fallback_policy_engine_validation` | reviewed | `whole_repo subtree` or package dependency | root package |
| `odiff` | `dmtrKovalenko/odiff` | `fallback_policy_engine_validation` | reviewed | `split_subdir subtree` | `npm_packages/odiff-bin` |
| `dom_to_pptx` | `atharva9167j/dom-to-pptx` | `visual_object_ir_normalizer` | reviewed | `whole_repo subtree` or package dependency | root package |
| `pptxgenjs` | `gitbrent/PptxGenJS` | `pptx_output_backend` | reviewed | `package dependency` | root package |
| `chartdetective` | `m-damien/ChartDetective` | `chart_semantic_extractor` | reviewed | `reference_only` | app-style repo |
| `table_transformer` | `microsoft/table-transformer` | `fake_table_detector` | reviewed | `reference_only` | research/model repo |
| `backstopjs` | `garris/BackstopJS` | `fallback_policy_engine_validation` | reviewed | `package dependency` or external CLI | root package + CLI |
| `opendataloader_pdf` | `opendataloader-project/opendataloader-pdf` | `semantic_ir_reference` | reviewed | `reference_only` now, `split_subdir subtree` later if needed | `java/opendataloader-pdf-core` for processor code |
| `docling` | `docling-project/docling` | `visual_object_ir_normalizer` | reviewed | `reference_only` now, package dependency for experiments | root Python package, optional `docling/` source subtree later |

---

## Repository notes

### 1. `pixelmatch`

#### Why it matters

Smallest deterministic pixel comparator among current candidates.

#### Working boundary

The real package boundary is the repository root.

- `package.json` uses `main = index.js`
- CLI is `bin/pixelmatch`
- tests are `tsc && node --test`

#### Decision

Prefer `whole_repo subtree` if local patching is needed.
Otherwise package dependency is also acceptable.

#### Current role in project

- validation backend core
- mismatch score engine
- diff image generator

---

### 2. `looks-same`

#### Why it matters

Alternative validation backend with perceptual comparison features.

#### Working boundary

The real package boundary is the repository root.

- `main = index.js`
- tests are mocha-based
- internal code lives under `lib/`

#### Decision

Good `whole_repo subtree` candidate, but less urgent than `pixelmatch`.

#### Current role in project

- secondary image diff engine
- perceptual comparison fallback

---

### 3. `odiff`

#### Why it matters

High-performance diff engine candidate.

#### Working boundary

The repository root is a private monorepo workspace and should **not** be subtree-imported as-is.
The real consumable boundary is:

```text
npm_packages/odiff-bin
```

That package wraps the binary and exposes the Node API.

#### Decision

Prefer `split_subdir subtree` for `npm_packages/odiff-bin`.
Do not use `whole_repo subtree` by default.

#### Current role in project

- high-speed validation backend option
- binary-backed image comparison wrapper

---

### 4. `dom-to-pptx`

#### Why it matters

Closest existing HTML/DOM-to-PPTX reference implementation among current candidates.

#### Working boundary

The official package boundary is the repository root.

- published outputs live under `dist/`
- build is Rollup-based from `src/index.js`
- test is Vitest
- CLI and `skills/` are part of the published package surface

Internally it contains identifiable subsystems:

- `src/index.js` orchestration
- `src/utils.js` rendering/text/table utilities
- `src/image-processor.js` image masking/object-fit logic
- `src/font-embedder.js` PPTX font embedding
- `src/pptx-normalizer.js` OOXML cleanup

#### Decision

Prefer `whole_repo subtree` if vendored.
`split_subdir = src/` is possible but does not align with upstream package boundaries.

#### Current role in project

- DOM measurement and mapping reference
- render queue / text collection / table extraction reference

---

### 5. `pptxgenjs`

#### Why it matters

The target PPTX serialization backend.

#### Working boundary

Root package.
But the repo is large and intentionally distributed as a package rather than a subtree-friendly source fragment.

#### Decision

Prefer `package dependency`.
Use subtree only if local patching of generator internals becomes unavoidable.

#### Current role in project

- PPTX compiler backend

---

### 6. `chartdetective`

#### Why it matters

Potential chart semantic extraction reference.

#### Working boundary

App-style React/TypeScript tool, not a clean library package.

#### Decision

Keep as `reference_only` for now.
Do not subtree-import until a narrower reusable module boundary is proven.

#### Current role in project

- chart understanding reference
- region/shape reconstruction ideas

---

### 7. `table_transformer`

#### Why it matters

Table detection and structure recognition reference.

#### Working boundary

Research/model repo with CLI-style Python scripts and model/config/data assumptions.
Not a small reusable package boundary.

#### Decision

Keep as `reference_only`.
If integrated later, treat as external model runtime rather than subtree-imported utility.

#### Current role in project

- fake table detector reference
- structure recognition reference

---

### 8. `backstopjs`

#### Why it matters

Full visual regression runner with browser automation and reporting.

#### Working boundary

Root package + CLI + report UI + config-driven workflow.
This is not a small image comparator package.

#### Decision

Prefer `package dependency` or external CLI usage.
Avoid subtree unless end-to-end local patching becomes necessary.

#### Current role in project

- optional full regression framework
- validation workflow reference

---

### 9. `opendataloader-pdf`

#### Why it matters

Strong reference for semantic schema, deterministic processor order, and semantic export generators.

#### Working boundary

The repository root is a workspace.
Immediate reuse targets are not the whole repo, but specific assets and subtrees:

- `schema.json`
- `options.json`
- `java/opendataloader-pdf-core`

#### Decision

Current policy:

- use `schema.json` and `options.json` as synced reference artifacts;
- keep repo `reference_only`;
- consider `split_subdir subtree` only for `java/opendataloader-pdf-core` if processor code must be patched locally.

#### Current role in project

- Visual Object IR schema reference
- policy surface reference
- deterministic processor-chain reference
- semantic writer-dispatch reference

---

### 10. `docling`

#### Why it matters

Strong reference for unified document representation, staged conversion pipeline, and plugin-based model selection.

#### Working boundary

The upstream consumable unit is the root Python package and CLI.
Potential code-interest boundaries include:

- `docling/document_converter.py`
- `docling/datamodel/document.py`
- `docling/pipeline/standard_pdf_pipeline.py`
- `docling/models/plugins/defaults.py`

#### Decision

Current policy:

- keep as `reference_only` for architecture guidance;
- use upstream package dependency for experiments;
- only consider subtree if local patching of pipeline internals becomes necessary.

#### Current role in project

- conversion entrypoint reference
- staged threaded pipeline reference
- unified document model reference
- plugin registry reference

---

## Immediate subtree priority

### Highest priority

1. `pixelmatch`
2. `odiff` → `npm_packages/odiff-bin`
3. `looks-same`

### Conditional priority

4. `dom-to-pptx`

### Package dependency priority

5. `pptxgenjs`
6. `backstopjs`

### Reference-only

7. `opendataloader-pdf`
8. `docling`
9. `table-transformer`
10. `chartdetective`

---

## Next action

Use this inventory together with:

- `docs/full_read_policy_v0.1.md`
- `third_party/subtrees.toml`

to drive actual import decisions.
