# general CI building blocks

Language-agnostic **composite actions**. Run them as steps inside your own job.

Because they are composite actions (not reusable workflows), the **caller's job**
owns `runs-on`, `permissions`, checkout, and secrets.

## `security`

Trivy filesystem scan; fails the job on matching vulnerabilities.

| input | default | notes |
|---|---|---|
| `severity` | `HIGH,CRITICAL` | comma-separated severities to fail on |
| `exit-code` | `1` | exit code on findings (`0` = report only) |

Requires the repo to be checked out first.

```yaml
jobs:
  security:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v7
      - uses: paolobellini/bellini.one/actions/general/security@v1.0
```

## `sbom`

Generates a CycloneDX SBOM with Trivy and uploads it to Dependency-Track.
Version is derived from the top entry in `CHANGELOG.md` (`## [x.y.z]`).

| input | required | notes |
|---|---|---|
| `project-name` | yes | Dependency-Track project name |
| `parent` | no | parent project UUID |
| `server-hostname` | yes | pass a secret |
| `api-key` | yes | pass a secret |

The caller's job must grant the permissions the upload needs:

```yaml
jobs:
  sbom:
    needs: deploy
    if: github.ref_name == 'main'
    runs-on: ubuntu-latest
    permissions:
      contents: read
      actions: read
      security-events: write
    steps:
      - uses: actions/checkout@v7
      - uses: paolobellini/bellini.one/actions/general/sbom@v1.0
        with:
          project-name: robobook.ai
          parent: 149fd4cc-b9c1-4ed6-a7a6-d57985cacf81
          server-hostname: ${{ secrets.DEPENDENCYTRACK_URL }}
          api-key: ${{ secrets.DEPENDENCYTRACK_API_KEY }}
```

## Consumer requirements

- Repository checked out before invoking (both actions scan `.`).
- `sbom` needs a `CHANGELOG.md` with a `## [x.y.z]` heading.
