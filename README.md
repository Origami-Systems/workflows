# Origami Systems Workflows

Shared GitHub Actions workflows for Origami Systems applications, services, and packages.

This repository centralizes CI/CD configuration across the Origami Systems organization, reducing duplicated configuration and keeping build, testing, and deployment processes consistent across repositories.

## Available workflows

| Workflow | Description |
|---|---|
| `go-build.yml` | Build and publish Go service Docker images to GHCR. |
| `go-lint.yml` | Run golangci-lint with private Go module authentication. |
| `nuxt-build.yml` | Build and publish production and staging Nuxt Docker images. |
| `nuxt-check.yml` | Prepare Nuxt and run Vite+ checks. |
| `astro-build.yml` | Build and deploy Astro websites to GitHub Pages. |
| `astro-check.yml` | Install dependencies and verify Astro builds using Vite+. |

## Maintenance

GitHub Actions dependencies are managed by Dependabot.

When modifying a reusable workflow:

- Preserve compatibility with existing calling repositories.
- Provide sensible defaults for new optional inputs.
- Request only the permissions required by the workflow.
- Never commit authentication credentials.
- Test changes before making them available to consuming repositories.

Changes to shared workflows may affect multiple Origami Systems repositories. Breaking changes should be documented and released under a new major version.

## About Origami Systems

This repository is maintained by [Origami Systems](https://origamisystems.org) as part of its shared development infrastructure.
