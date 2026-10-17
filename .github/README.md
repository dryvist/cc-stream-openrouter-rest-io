# `.github/` — workflows for this pack

This repository calls pinned reusable workflows from `jpe-cribl/.github`.
Caller workflows in this repo (`test.yml`, `release.yml`, `e2e.yml`) wire them
up to GitHub events.

## Installation

The caller workflows under `.github/workflows/` reference the shared workflows
at `jpe-cribl/.github/.github/workflows/<file>@<40-hex-sha> # main`. The shared
test workflow checks the shared scripts out to `.gh-shared` for both its
validate and test jobs; the shared release workflow uses those scripts from its
own checkout.

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

The pack test, release, and e2e workflows are reusable workflows hosted in
[`jpe-cribl/.github`](https://github.com/jpe-cribl/.github)
(`cribl-pack-test.yml`, `cribl-pack-release.yml`, `cribl-pack-e2e.yml`); their
inputs and jobs are documented there.

## Caller workflows in this repo

| File | Trigger | Calls |
| --- | --- | --- |
| `test.yml` | path-filtered PR + main push | central pack-test |
| `release.yml` | main push + PR | central release + pack assets |
| `e2e.yml` | manual dispatch + e2e path-filtered PR | shared cribl-pack-e2e |

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
