# bellini.one

Central, versioned CI/CD building blocks shared across `paolobellini` projects:
reusable GitHub Actions **workflows** and **composite actions**.

Consumer repos reference these instead of copy-pasting pipeline YAML, so a fix
made here propagates to every project on the next tag bump.

## Structure

```
.
├── .github/
│   ├── workflows/              # reusable workflows (must live at repo root)
│   │   ├── laravel-test.yml    # phpunit/pest + DB service (mysql/pgsql)
│   │   └── laravel-lint.yml    # pint + rector + phpstan + node checks
│   └── dependabot.yml
├── actions/                    # composite actions (callable from sub-folders)
│   ├── general/                # language-agnostic actions
│   │   ├── sbom/               # cyclonedx sbom -> dependency-track
│   │   └── security/           # trivy filesystem vulnerability scan
│   └── laravel/                # laravel-specific actions
│       └── setup-app/          # php + node + composer install & cache
├── learnings/                  # notes / docs (placeholder)
├── stack/                      # stack references (placeholder)
└── templates/                  # project templates (placeholder)
```

### Why two mechanisms?

GitHub only discovers **reusable workflows** at the repository root
(`.github/workflows/`) — sub-folders are ignored. **Composite actions** have no
such limit and can live anywhere, so they are referenced by path.

| | reusable workflow | composite action |
|---|---|---|
| location | root `.github/workflows/` only | any sub-folder |
| runs as | its own job | a step inside the caller's job |
| can define | `services`, `permissions`, `environment`, matrix | steps only |
| referenced by | `owner/repo/.github/workflows/file.yml@ref` | `owner/repo/path@ref` |

Workflows that need a job-level feature (e.g. the test workflow's DB
`services:`) stay as reusable workflows. Everything else is a composite action.

## Versioning

Everything in this repo shares a **single tag**. Consumers pin that tag:

```yaml
uses: paolobellini/bellini.one/.github/workflows/laravel-test.yml@v1.0
uses: paolobellini/bellini.one/actions/general/security@v1.0
```

Current release: **`v1.0`**.

- Bump the tag when you change any workflow/action.
- A tag is a whole-repo snapshot; unchanged files are unaffected, and consumers
  stay on their pinned tag until they bump.
- For maximum supply-chain safety pin a commit SHA instead of a moving tag.

## Usage

- Laravel workflows → [`actions/laravel/README.md`](actions/laravel/README.md)
- General actions → [`actions/general/README.md`](actions/general/README.md)
