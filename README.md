# daemonless/workflows

The reusable GitHub Actions workflow that builds, tests, signs and publishes
the [daemonless](https://daemonless.io) FreeBSD container images.

## Calling it

An image repo's `.github/workflows/build.yaml`:

```yaml
name: Build FreeBSD Container

on:
  push:
    branches: [main]
    paths-ignore: ['*.md', 'LICENSE', '.gitignore']
  pull_request:
    branches: [main]
  workflow_dispatch:

jobs:
  build:
    uses: daemonless/workflows/.github/workflows/daemonless-build.yaml@v1
    with:
      image_name: radarr
    secrets: inherit
```

## Inputs

| input | default | what |
|---|---|---|
| `image_name` | (required) | image name, e.g. `radarr` |
| `variant` | all | build one variant (tag) only |
| `pre_artifact_name` | | artifact to download before the build |
| `pre_artifact_path` | | where to download it |
| `registry` | `ghcr.io/daemonless` | registry prefix to push to |
| `build_tools_ref` | latest release | build-tools tag |
| `generate` | `true` | run `dbuild generate` first |
| `arch` | org default | build one architecture only |
| `dbuild_ref` | `DBUILD_REF` | dbuild branch, tag or SHA; testing only |
| `linux` | `false` | also build from `Containerfile.linux` |
| `build_timeout` | `180` | minutes before a build job is cancelled |

## Versions

- `v1` is what the fleet calls. It only moves forward along `main`; a
  breaking input change becomes `v2`.
- `DBUILD_REF` in `daemonless-build.yaml` is the dbuild release the workflow
  runs. Moving to a new dbuild is a PR that bumps it.

## Canaries

These images call `@main`, so a change is built before `v1` moves:

| image | covers |
|---|---|
| radarr | common service, .NET annotation |
| reverse-proxy | `linux: true` |
| bulwark-webmail | `pre_artifact_name` / `pre_artifact_path` |
| immich | stack with choices, compose only |

## Releasing a change

1. Merge the PR to `main` (a workflow change, or a `DBUILD_REF` bump).
2. Build the canaries:

   ```sh
   for r in radarr reverse-proxy bulwark-webmail immich; do
     gh workflow run build.yaml --repo daemonless/$r
   done
   ```

3. When all four succeed, move `v1`:

   ```sh
   git switch main && git pull
   git tag -f v1
   git push -f origin v1
   ```

The actions under `.github/actions/` are called `@v1`, so a change to them
reaches the canaries only after `v1` moves.

History before this repo is in
[daemonless/dbuild](https://github.com/daemonless/dbuild) (copied from
de592fb).
