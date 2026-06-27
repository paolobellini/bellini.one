# laravel CI building blocks

Shared CI for Laravel projects: two reusable workflows + one composite action.

> The reusable **workflows** live at the repo root in
> [`.github/workflows/`](../../.github/workflows) (GitHub only discovers
> reusable workflows there). The **composite action** lives here.

## Reusable workflows

### `laravel-test.yml`

Runs the test suite (`composer tests`) against a database service.

Inputs (all optional):

| input | default | notes |
|---|---|---|
| `php-version` | `8.4` | |
| `node-version` | `24` | |
| `database` | `mysql` | Laravel `DB_CONNECTION` (`mysql`, `pgsql`) |
| `db-image` | `mysql:8.0` | service container image |
| `db-port` | `3306` | host + container port (engine-native) |
| `db-options` | mysql health-check | container options; health-cmd differs per engine |

```yaml
jobs:
  test:                       # defaults -> mysql
    uses: paolobellini/bellini.one/.github/workflows/laravel-test.yml@v1.0

  test-pgsql:
    uses: paolobellini/bellini.one/.github/workflows/laravel-test.yml@v1.0
    with:
      database: pgsql
      db-image: postgres:16
      db-port: '5432'
      db-options: >-
        --health-cmd="pg_isready"
        --health-interval=10s
        --health-timeout=5s
        --health-retries=5
```

Credentials are fixed to `root` / `root`, database `testing`.

### `laravel-lint.yml`

Runs `composer php-checks` (pint + rector dry-run + phpstan) and
`composer node-checks`.

| input | default |
|---|---|
| `php-version` | `8.4` |
| `node-version` | `24` |

```yaml
jobs:
  lint:
    uses: paolobellini/bellini.one/.github/workflows/laravel-lint.yml@v1.0
```

## Composite action

### `setup-app`

Installs PHP + Node, copies `.env`, caches and installs Composer + npm deps,
builds assets. Used internally by the workflows above; also usable directly.

| input | required |
|---|---|
| `php-version` | yes |
| `node-version` | yes |

```yaml
steps:
  - uses: actions/checkout@v7
  - uses: paolobellini/bellini.one/actions/laravel/setup-app@v1.0
    with:
      php-version: '8.4'
      node-version: '24'
```

## Consumer requirements

- `composer tests`, `composer php-checks`, `composer node-checks` scripts.
- `.env.example` and `composer.lock` present.
