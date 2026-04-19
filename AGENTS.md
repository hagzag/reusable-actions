# AGENTS.md

This file provides guidance to coding agent when working with code in this repository.

## purpose

Reusable GitHub Actions `workflow_call` workflows. All CI runs inside `ghcr.io/hagzag/tools` — no toolchain install steps in any workflow.

## workflows

| File | Trigger | What it does |
|------|---------|--------------|
| `.github/workflows/release.yml` | `workflow_call` | Runs `semantic-release` inside tools image; writes a default `.releaserc.yml` if none exists |
| `.github/workflows/docker-build.yml` | `workflow_call` | Multi-arch Docker build (amd64+arm64), registry cache, optional `grype` scan |

## architecture constraints

- **`container.image` cannot use `${{ env.* }}`** — GitHub Actions does not evaluate the `env` context inside `container.image`. Always hardcode or use `inputs.*` (which is allowed).
- **`inputs.*` in `container.image` is valid** — callers can pin a specific tools tag via `tools-image` input.
- **Registry cache pattern** — `docker-build.yml` uses `type=registry,ref=<image>:buildcache` (not GHA cache) to avoid the 5 GB eviction limit.
- **Scan job runs in tools container** — `grype` is pre-installed in `ghcr.io/hagzag/tools`; no install step needed.
- **Default `.releaserc.yml` written on the fly** — `release.yml` checks for `.releaserc.yml`, `.releaserc.json`, `release.config.js` in that order. Only writes the default when none exist.

## reference configs

| File | Purpose |
|------|---------|
| `.releaserc.yml` | Default GitHub release config (commit-analyzer, release-notes-generator, changelog, git, exec, github) |
| `.releaserc-gitlab.yml` | GitLab variant (swap `@semantic-release/github` → `@semantic-release/gitlab`) |

## adding a new reusable workflow

1. Add the file under `.github/workflows/`.
2. Use `on: workflow_call` with typed `inputs` and `secrets`.
3. Use `ghcr.io/hagzag/tools` as the container image (accept it as an `inputs.tools-image` string with that default).
4. Document it in `README.md` with an inputs/secrets/outputs table.
5. Add a row to the workflows table above.

## semantic-release plugins available in tools image

`commit-analyzer`, `release-notes-generator`, `changelog`, `git`, `exec`, `github`, `gitlab`

All plugins are globally installed — no `npm install` step needed in any workflow.
