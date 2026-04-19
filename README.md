# reusable-actions

Reusable GitHub Actions workflows. All CI runs inside [`ghcr.io/hagzag/tools`](https://github.com/hagzag/tools) — no toolchain installation needed.

## workflows

| Workflow | File | Purpose |
|----------|------|---------|
| `release` | [`release.yml`](.github/workflows/release.yml) | semantic-release via tools image |
| `docker-build` | [`docker-build.yml`](.github/workflows/docker-build.yml) | multi-arch Docker build + grype scan |

---

## release

Runs `semantic-release` inside `ghcr.io/hagzag/tools`. If the calling repo has no `.releaserc.yml`, a default GitHub-flavoured config is written on the fly. Existing configs are respected.

### usage

```yaml
jobs:
  release:
    uses: hagzag/reusable-actions/.github/workflows/release.yml@main
    secrets:
      github-token: ${{ secrets.GITHUB_TOKEN }}
```

### inputs

| Input | Default | Description |
|-------|---------|-------------|
| `tools-image` | `ghcr.io/hagzag/tools:latest` | tools image tag |
| `runs-on` | `ubuntu-latest` | runner label |

### secrets

| Secret | Required | Description |
|--------|----------|-------------|
| `github-token` | yes | token with `contents:write` and `packages:read` |

### default `.releaserc.yml`

Written when no release config is found in the repo root:

```yaml
branches:
  - main
  - name: dev
    prerelease: true
plugins:
  - "@semantic-release/commit-analyzer"
  - "@semantic-release/release-notes-generator"
  - "@semantic-release/changelog"
  - "@semantic-release/git"
  - - "@semantic-release/exec"
    - verifyReleaseCmd: "echo ${nextRelease.version} > nextVersion"
  - "@semantic-release/github"
```

To override: commit your own `.releaserc.yml`, `.releaserc.json`, or `release.config.js` to the repo root. See [`.releaserc.yml`](.releaserc.yml) and [`.releaserc-gitlab.yml`](.releaserc-gitlab.yml) in this repo for examples.

### conventional commits

```
feat: add feature       → minor bump
fix: correct bug        → patch bump
feat!: breaking change  → major bump
```

---

## docker-build

Builds a multi-arch image (`linux/amd64` + `linux/arm64`), pushes to GHCR, and optionally runs a `grype` vulnerability scan. Uses registry-based layer cache to speed up repeated builds.

### usage

```yaml
jobs:
  docker:
    uses: hagzag/reusable-actions/.github/workflows/docker-build.yml@main
    secrets:
      github-token: ${{ secrets.GITHUB_TOKEN }}
```

### inputs

| Input | Default | Description |
|-------|---------|-------------|
| `tools-image` | `ghcr.io/hagzag/tools:latest` | tools image used in scan step |
| `runs-on` | `ubuntu-latest` | runner label |
| `image-name` | `ghcr.io/<owner>/<repo>` | override target image name |
| `context` | `.` | Docker build context |
| `dockerfile` | `Dockerfile` | path to Dockerfile |
| `platforms` | `linux/amd64,linux/arm64` | target platforms |
| `scan` | `true` | run grype scan after build |
| `fail-on` | `high` | minimum severity to fail scan |

### secrets

| Secret | Required | Description |
|--------|----------|-------------|
| `github-token` | yes | token with `contents:write` and `packages:write` |

### outputs

| Output | Description |
|--------|-------------|
| `image` | full image reference pushed (name:tag) |
| `digest` | image digest (`sha256:...`) |

### grype scan policy

If a `.grype.yaml` exists in the repo root it is passed to grype automatically. Use it to suppress known-unfixable CVEs:

```yaml
# .grype.yaml
ignore:
  - package:
      type: go-module      # compiled-in, not patchable via apk
  - vulnerability: CVE-XXXX-YYYY
    package:
      name: some-pkg
      type: apk
```

Set `scan: false` to skip the scan entirely, or `fail-on: critical` to only fail on critical findings.

---

## examples in this repo

| File | Purpose |
|------|---------|
| [`.releaserc.yml`](.releaserc.yml) | Default GitHub release config |
| [`.releaserc-gitlab.yml`](.releaserc-gitlab.yml) | GitLab release config example |
