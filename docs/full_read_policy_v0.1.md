# Full Read Policy v0.1

This document defines the default repository ingestion policy for CLI-based agent analysis in `html-to-editable-pptx`.

## Goal

When a repository is selected for reuse analysis, the agent must read the full checked-out source tree before making architectural or subtree import decisions.

Full Read applies to:

- the superproject;
- all initialized submodules;
- all tracked text files in the current checkout.

This policy exists because module-boundary analysis, subtree planning, and adapter design are unreliable when performed from partial checkout state.

## Default clone mode

Use shallow full clone with recursive submodules.

```bash
git clone --depth 1 --single-branch --no-tags \
  --recurse-submodules --shallow-submodules \
  <repo-url> <dest>
```

### Why this is the default

- `--depth 1` reduces history cost while preserving the complete current file tree.
- `--single-branch` prevents unrelated branch history from inflating the clone.
- `--no-tags` avoids downloading tag graph noise during read-only analysis.
- `--recurse-submodules` ensures the readable worktree includes nested source dependencies.
- `--shallow-submodules` keeps submodules readable while still bounded to the current state.

## Disallowed clone modes

The following options are disallowed for Full Read work:

- `--sparse`
- `--filter=blob:none`
- any sparse-checkout workflow
- any partial clone workflow that omits file contents at bootstrap time

These modes make module-boundary analysis incomplete because they do not materialize the full tracked text tree up front.

## Read target definition

A file is part of the Full Read target when:

1. it is tracked by Git in the current checkout;
2. it belongs to either the superproject or an initialized submodule;
3. it is a text file or symlink.

Binary files are not required read targets.

## Standard read order

The agent should process files in deterministic order:

1. repository root metadata
   - `README*`
   - package/build manifests
   - repo-level config files
2. entrypoints
   - package roots
   - CLI entrypoints
   - exported library entrypoints
3. core source tree
4. tests
5. examples, demos, fixtures
6. docs and architectural references
7. submodules, each under the same policy

## Coverage gate

No downstream decision is considered valid until all read targets are accounted for.

Required artifacts:

```text
.fullread/
  manifest.tsv
  done.tsv
  root.head
  submodules.status
  unread.tsv
```

### Completion rule

`Full Read complete` means:

- every `(scope, path)` in `manifest.tsv` appears in `done.tsv`;
- `unread.tsv` is empty;
- root HEAD matches `root.head`;
- recursive submodule status matches `submodules.status`.

## Operational scripts

The standard workflow is:

1. `tools/full-read-bootstrap.sh`
2. `tools/full-read-build-manifest.sh`
3. iterative file reading + `tools/full-read-mark.sh`
4. `tools/full-read-verify.sh`

## Decision policy after Full Read

Only after the coverage gate passes may the agent decide one of the following import modes:

- `whole_repo subtree`
- `split_subdir subtree`
- `package dependency`
- `reference_only`
- `reject`

## Notes for subtree planning

Full Read is not the same as subtree adoption.

- Full Read answers: `what is the real working module boundary?`
- subtree planning answers: `what should be vendor-imported into this repository?`

The default assumption is conservative:

- adopt runtime-critical, small, stable packages with clear entrypoints;
- keep schema-only, policy-only, or large research repos as `reference_only` until a narrower module boundary is proven.
