# `.github/` — workflows for this template

Reusable GitHub Actions workflows are callable from any dryvist Cribl pack.
Caller workflows in this repo (`test.yml`, `release.yml`) wire them up to
GitHub events.

## Installation

These workflows install themselves implicitly when a new pack scaffolds from
`dryvist/cc-edge-pack-template` (`gh repo create --template`). The caller
workflows under `.github/workflows/` are copied verbatim; the reusable
workflows they reference resolve at runtime against
`dryvist/cc-edge-pack-template/.github/workflows/<file>@<40-hex-sha> # vX.Y.Z`.

If you're adding a workflow to a non-template repo manually, copy the relevant
caller from this directory into your repo's `.github/workflows/` and adjust
the `uses:` ref to pin a different template release. Pack-local reusable
workflows are pinned to their commit SHA; the shared workflow homes use `@v1`.

## Usage

Once installed, the workflows trigger automatically:

- Open a PR touching `default/`, `data/samples/`, `tests/`, `scripts/`, or
  `package.json` → `test.yml` runs validate + Vitest.
- Push to `main` → `release.yml` calls central `_release.yml` to open or update
  the release PR.
- Repository-specific asset and publish jobs remain in `release.yml` and call
  the pack release workflow as needed.

No manual invocation is needed. Override defaults by editing the caller
workflow's `with:` block (e.g., pass a custom `cribl_versions` JSON array to
test specific upstream Cribl releases).

## Reusable workflows

The pack test and release workflows are reusable workflows hosted in
[`dryvist/cc-edge-pack-template`](https://github.com/dryvist/cc-edge-pack-template)
(`cribl-pack-test.yml`, `cribl-pack-release.yml`); their inputs and jobs
are documented there.

## Caller workflows in this repo

| File | Trigger | Calls |
| --- | --- | --- |
| `test.yml` | path-filtered PR + main push | central pack-test |
| `release.yml` | main push + PR | central release + pack assets |

## Updating reusable workflow inputs

Adding/renaming an input is a breaking change for every consumer pack. Use the
`workflow_call.inputs.<name>.default` field to keep the change backward
compatible whenever possible. Bump major version of the workflow only if you
absolutely cannot.

## Pinning external actions

Per [`SECURITY.md`](https://github.com/JacobPEvans/.github/blob/main/SECURITY.md):

- Trusted (`actions/*`, `pnpm/action-setup`, `softprops/action-gh-release`):
  semver tag pins (`@v4`, etc.)
- Untrusted (everything else): SHA pins. Renovate (per the org's
  `renovate.json`) auto-converts version tags to SHA on first run.
