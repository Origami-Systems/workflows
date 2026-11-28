# Origami Systems Workflows

Reusable GitHub Actions workflows for Origami Systems applications and services. Callers must grant the permissions listed below; a reusable workflow cannot increase the permissions granted by its caller.

Existing callers use `@main`. For predictable releases, callers should pin this repository to a reviewed commit SHA. Publish a new major tag before making breaking changes, document its migration, and move callers deliberately. The current defaults remain compatible with existing callers; in particular, image builds push by default.

## Nuxt

`nuxt-check.yml` installs dependencies, prepares Nuxt, and runs `vp check`. It requires `contents: read` and `packages: read`. `runner` defaults to `ubuntu-latest`; `working-directory` defaults to `.`. Vite+ resolves Node.js from the project. The working directory is also where setup detects the lockfile and installs dependencies. For workspaces that must install at the root, use the root directory and run project-specific checks in the caller.

```yaml
jobs:
  check:
    permissions:
      contents: read
      packages: read
    uses: Origami-Systems/workflows/.github/workflows/nuxt-check.yml@main
    with:
      working-directory: .
```

`nuxt-build.yml` builds a Docker image and optionally pushes to GHCR. It requires `contents: read` and `packages: write`, including when `push` is false under the current permission contract. The caller's `GITHUB_TOKEN` needs access to any private packages used during the Docker build. `SENTRY_AUTH_TOKEN` is optional and passed as a BuildKit secret. Inputs: required `image`; `tag` defaults to `latest`; `push` defaults to `true`; `dockerfile` defaults to `Dockerfile.production`; `context` defaults to `.`; `platforms` defaults to `linux/amd64`; and `runner` defaults to `ubuntu-latest`. It tags the image with both `tag` and the commit SHA.

```yaml
jobs:
  build:
    permissions:
      contents: read
      packages: write
    uses: Origami-Systems/workflows/.github/workflows/nuxt-build.yml@main
    with:
      image: example-app
      push: false
      tag: test
```

For a release, explicitly set `push: true` and the intended `tag`. Restrict the caller's release trigger to trusted branches or tags. For pull request builds, select an explicit PR tag if pushing is required, or set `push: false`.

## Go

`go-lint.yml` runs golangci-lint and needs `contents: read`. `go-build.yml` builds a container and requires `contents: read` and `packages: write`, even for `push: false` under the current contract. Both require the caller's `CI_APP_PRIVATE_KEY` secret and `CI_APP_CLIENT_ID` variable for a GitHub App installed with read access to `go-shared`. The build passes its App token to Docker as the `github_token` and `ORIGAMI_SYSTEMS_GO_PACKAGES` BuildKit secrets. Dockerfiles can consume either name and must not copy the token into an image layer.

```yaml
jobs:
  lint:
    permissions:
      contents: read
    uses: Origami-Systems/workflows/.github/workflows/go-lint.yml@main
    secrets:
      CI_APP_PRIVATE_KEY: ${{ secrets.CI_APP_PRIVATE_KEY }}
  build:
    permissions:
      contents: read
      packages: write
    uses: Origami-Systems/workflows/.github/workflows/go-build.yml@main
    with:
      image: example-service
      push: false
      publish-latest: false
    secrets:
      CI_APP_PRIVATE_KEY: ${{ secrets.CI_APP_PRIVATE_KEY }}
```

Go lint inputs are `go-version` (default `1.27.3`), `lint-version` (default `v2.15.0`), `runner` (default `ubuntu-24.04`), and `working-directory` (default `.`). Go build requires an `image` and a root `VERSION` file. Its other inputs are `tag` (default empty), `platforms` (default `linux/amd64,linux/arm64`), `dockerfile` (default `Dockerfile.production`), `context` (default `.`), `push` (default `true`), and `publish-latest` (default `true`). When `tag` is set, the image uses that tag and the commit SHA. Otherwise it uses `VERSION` and the commit SHA, plus `latest` when `publish-latest` is true. For pull requests, pass `tag: ${{ format('pr-{0}', github.event.pull_request.number) }}` to avoid overwriting release tags. Choose release triggers carefully because version and latest tags can be overwritten.

## Astro

`astro-check.yml` installs with Vite+ and runs a build command. It requires `contents: read` and `packages: read`. Inputs are `runner` (default `ubuntu-latest`), `node-version` (default `26.13.1`), `working-directory` (default `.`), and `build-command` (default `vp run build`). The setup action installs and detects the lockfile in `working-directory`.

```yaml
jobs:
  check:
    permissions:
      contents: read
      packages: read
    uses: Origami-Systems/workflows/.github/workflows/astro-check.yml@main
```

`astro-build.yml` builds with the Astro action and deploys to GitHub Pages by default. It requires `contents: read`, `packages: read`, `pages: write`, and `id-token: write`; configure the caller repository's Pages source to use GitHub Actions. Inputs are `runner`, `node-version`, `package-manager` (default `pnpm@12.23.0`), `build-command` (default `pnpm run build`), `path` (default `.`), and `deploy` (default `true`).

```yaml
jobs:
  pages:
    permissions:
      contents: read
      packages: read
      pages: write
      id-token: write
    uses: Origami-Systems/workflows/.github/workflows/astro-build.yml@main
    with:
      deploy: true
```

## Maintenance

Dependabot maintains GitHub Actions dependencies. Readable action references are pinned to commits in `.github/workflows/actions.lock`, including dependencies used by composite actions. Review lockfile changes, including commit and repository identity changes, with each dependency update. The actionlint version in `lint.yml` is pinned separately and needs manual updates.

Install GitHub's extension with `gh extension install github/gh-actions-lock`, then run `gh actions-lock --no-narrow` whenever workflows gain or lose action dependencies. Run `gh actions-lock --verify-local` for an offline coverage check or `gh actions-lock --verify` to check the pins against GitHub. Dependabot updates the lockfile alongside action version updates for onboarded workflows. Reusable workflow calls are not pinned by this lockfile; external callers should pin their reusable workflow references separately.

The integration workflow calls the frontend checks against fixtures, including a nested Nuxt project. It does not exercise private package access, Docker publishing, or Pages deployment. Test those paths in an appropriate caller before releasing related changes. Changes to the working directory or default publishing behavior need a caller audit and a documented migration. A future major version can make image publishing and mutable tags explicit after callers have opted in.
