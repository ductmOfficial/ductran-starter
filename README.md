# ductran-starter

[![CI](https://github.com/ductmOfficial/ductran-starter/actions/workflows/ci.yml/badge.svg)](https://github.com/ductmOfficial/ductran-starter/actions/workflows/ci.yml)

A TypeScript monorepo template for my portfolio projects. Every project starts from the same foundation: strict TypeScript, shared lint and format configs, component tests, CI on every pull request, a local PostgreSQL, and enforced commit conventions.

## What's inside

| Area        | Tools                                                                                |
| ----------- | ------------------------------------------------------------------------------------ |
| Monorepo    | [pnpm](https://pnpm.io) workspaces, [Turborepo](https://turborepo.com)               |
| Language    | TypeScript 6 (strict), Node.js 24 LTS                                                |
| Lint/format | ESLint 10 (flat config, type-aware `typescript-eslint`, `@eslint-react`), Prettier 3 |
| Testing     | Vitest 5, jsdom, Testing Library, jest-dom, user-event, v8 coverage                  |
| UI sample   | React 19 component library (`@repo/ui`), Vite library mode + `tsc` declarations      |
| Database    | PostgreSQL 18 via Docker Compose                                                     |
| CI          | GitHub Actions, protected `main` branch (ruleset), Dependabot                        |
| Conventions | Conventional Commits (commitlint + husky), PR template, issue forms                  |

## Structure

```
.
├── apps/                       # applications (empty in the template)
├── packages/
│   ├── eslint-config/          # @repo/eslint-config: `base`, `react`, `next`, `vite` flat configs
│   ├── prettier-config/        # @repo/prettier-config
│   ├── typescript-config/      # @repo/typescript-config: base, node, react-library, nextjs, vite-app
│   └── ui/                     # @repo/ui: sample React library (Button + tests)
├── .github/
│   ├── workflows/ci.yml        # CI for pull requests and pushes to main
│   ├── workflows/pr-title.yml  # re-checks the PR title when only the title is edited
│   ├── dependabot.yml          # weekly grouped npm and GitHub Actions updates
│   ├── rulesets/protect-main.json
│   ├── pull_request_template.md
│   └── ISSUE_TEMPLATE/         # bug report and feature request forms
├── .husky/commit-msg           # runs commitlint on every commit
├── compose.yaml                # local PostgreSQL 18
├── turbo.json                  # task pipeline
├── pnpm-workspace.yaml
└── PROGRESS.md                 # build log: steps, decisions, issues
```

## Setup

### 1. Prerequisites

| Tool                   | Version                    | Needed for        | Install                                                        |
| ---------------------- | -------------------------- | ----------------- | -------------------------------------------------------------- |
| Node.js                | 24 (`.nvmrc`)              | everything        | `nvm install && nvm use` (or `fnm use`)                        |
| pnpm                   | 12.14.0 (`packageManager`) | everything        | `corepack enable` (or `npm install -g pnpm@12.14.0`)           |
| Git                    | any recent                 | everything        |                                                                |
| Docker with Compose v2 | any recent                 | the database only | [Docker Desktop](https://docs.docker.com/desktop/) or OrbStack |

`pnpm install` refuses other Node.js majors (`engineStrict`). Corepack ships with Node.js 24: after `corepack enable`, the first `pnpm` command downloads the exact version from `packageManager` (answer `Y` if it asks).

There are no browser tests in the template, so Playwright is not needed.

### 2. Clone and install

```bash
git clone https://github.com/ductmOfficial/ductran-starter.git
cd ductran-starter
nvm use
corepack enable
pnpm install          # also installs the commit-msg hook
```

### 3. Configure (optional)

Only the local database reads environment variables, and `compose.yaml` already has the same defaults, so this step is optional:

```bash
cp .env.example .env
```

`.env` and `.env.*` are ignored by git; only `.env.example` is committed.

| Variable            | Required | Default                                             | What it does                                                      |
| ------------------- | -------- | --------------------------------------------------- | ----------------------------------------------------------------- |
| `POSTGRES_USER`     | no       | `postgres`                                          | Superuser created in the container                                |
| `POSTGRES_PASSWORD` | no       | `postgres`                                          | Its password (local development only)                             |
| `POSTGRES_DB`       | no       | `app`                                               | Database created on first start                                   |
| `POSTGRES_PORT`     | no       | `5432`                                              | Port published on `127.0.0.1`                                     |
| `DATABASE_URL`      | no       | `postgresql://postgres:postgres@localhost:5432/app` | Connection string for apps; keep it in sync with the values above |

These credentials are for the container on your machine; never reuse them in production.

## Running

### Build the packages

```bash
pnpm build
```

The template has no app yet, so there is no dev server. `pnpm dev` runs every package's `dev` task; in `@repo/ui` that is `vite build --watch`, which rebuilds `dist/` on every change. Stop it with `Ctrl+C`.

### Local database

```bash
pnpm db:up            # start PostgreSQL and wait until it is healthy
pnpm db:psql          # open psql inside the container
pnpm db:psql -c "select version();"   # extra arguments are passed through
pnpm db:down          # stop it, keep the data
```

Connection string: `postgresql://postgres:postgres@localhost:5432/app` (development only). There are no migrations or seed data in the template.

## Testing

| Command                             | What it runs                                               |
| ----------------------------------- | ---------------------------------------------------------- |
| `pnpm test`                         | Vitest in every package                                    |
| `pnpm test:coverage`                | Tests with coverage; HTML report in `packages/*/coverage/` |
| `pnpm lint` / `pnpm typecheck`      | ESLint (warnings fail) and `tsc --noEmit` everywhere       |
| `pnpm format:check` / `pnpm format` | Prettier check / write                                     |
| `pnpm build`                        | Build every package                                        |

To run what CI runs before opening a pull request:

```bash
pnpm install --frozen-lockfile
pnpm format:check
pnpm turbo run lint typecheck test build
```

Run a script in one package with a filter, e.g. `pnpm --filter @repo/ui test:watch`.

### All scripts

Run from the repository root.

| Script               | What it does                                                     |
| -------------------- | ---------------------------------------------------------------- |
| `pnpm build`         | Build every package (`turbo run build`)                          |
| `pnpm dev`           | Every package's `dev` task (`turbo run dev`)                     |
| `pnpm lint`          | ESLint in every package; warnings fail the run                   |
| `pnpm typecheck`     | `tsc --noEmit` in every package                                  |
| `pnpm test`          | Vitest in every package                                          |
| `pnpm test:coverage` | Tests with coverage; HTML report in `packages/*/coverage/`       |
| `pnpm format`        | Format the whole repo with Prettier                              |
| `pnpm format:check`  | Check formatting without writing (used by CI)                    |
| `pnpm db:up`         | Start PostgreSQL in the background and wait until it is healthy  |
| `pnpm db:down`       | Stop PostgreSQL, keep the data                                   |
| `pnpm db:reset`      | Stop PostgreSQL and delete all data                              |
| `pnpm db:logs`       | Follow the PostgreSQL logs                                       |
| `pnpm db:psql`       | Open `psql` in the container; extra arguments are passed through |

## Troubleshooting

| Symptom                                                      | Fix                                                                                                               |
| ------------------------------------------------------------ | ----------------------------------------------------------------------------------------------------------------- |
| `pnpm install` fails with `ERR_PNPM_UNSUPPORTED_ENGINE`      | Wrong Node.js major: `nvm use` (Node.js 24), then install again                                                   |
| `pnpm: command not found`, or a different pnpm version runs  | `corepack enable`, then open a new terminal; `pnpm -v` should print `12.14.0`                                     |
| `pnpm db:up` fails with "port is already allocated"          | Another PostgreSQL uses 5432: set `POSTGRES_PORT=5433` in `.env` and change the port in `DATABASE_URL`            |
| `pnpm db:up` cannot connect to the Docker daemon             | Start Docker Desktop (or OrbStack) first                                                                          |
| A machine with little memory kills the build (exit code 137) | Run fewer tasks at once: `pnpm turbo run lint typecheck test build --concurrency=2`                               |
| An app cannot resolve `@repo/ui`                             | Run `pnpm build` once: packages are imported from their `dist/` output                                            |
| The commit is rejected                                       | The hook enforces [Conventional Commits](https://www.conventionalcommits.org): `feat(ui): …`, `fix: …`, `docs: …` |

## How it works

**Task pipeline.** Each package defines its own `build`, `lint`, `typecheck` and `test` scripts. Root scripts call `turbo run <task>`, which runs them in dependency order (`^build` means "build my dependencies first"), in parallel where possible, and caches results by content hash. A second run with no changes finishes in milliseconds (`FULL TURBO`).

**Shared configuration.** TypeScript, ESLint and Prettier configs live in `packages/*-config` and are consumed with `workspace:*`. A new package only needs a two-line `eslint.config.js` and a `tsconfig.json` that `extends` a shared preset.

**Internal packages are built.** `@repo/ui` is built with Vite library mode (one ES module per source file, so apps can drop unused code), and `tsc` adds only the type declarations. Vite empties `dist/` first, so renamed or deleted files never linger. `exports` point at `dist/`. `tsconfig.json` (editor and typecheck) includes tests; `tsconfig.build.json` excludes them from the output.

**Tests.** Vitest runs in jsdom. Component tests use Testing Library queries by role and `user-event`, so they check behavior the way a user sees it. Globals are off: test functions are imported explicitly and `vitest.setup.ts` calls `cleanup()`.

**CI.** `.github/workflows/ci.yml` installs with a frozen lockfile, lints commit messages and the PR title, checks formatting, then runs `turbo run lint typecheck test build`. The pnpm store and the Turbo cache are cached between runs, and a new push cancels the previous run of the same pull request. Editing only the PR title re-runs a light title check (`pr-title.yml`). Actions are pinned to commit SHAs, Dependabot proposes updates, and the token is read-only.

**Protected `main`.** The ruleset in `.github/rulesets/protect-main.json` requires a pull request, a passing `Verify` check, an up-to-date branch and linear history, and blocks force pushes and deletion. Merge with **rebase** (keeps focused commits) or **squash** (one commit from the PR title).

## Contributing workflow

1. Branch from `main`: `git switch -c feat/short-description`
2. Commit using [Conventional Commits](https://www.conventionalcommits.org): `type(scope): summary`, e.g. `feat(ui): add Card component`. The `commit-msg` hook rejects anything else.
3. Push and open a PR: `gh pr create`
4. Wait for the `Verify` check: `gh pr checks --watch`
5. Merge: `gh pr merge --rebase --delete-branch`

## Using this template for a new project

1. Create the repository: `gh repo create <name> --template ductmOfficial/ductran-starter --public --clone`
2. Rename the project: `name` in the root `package.json`, `name` in `compose.yaml`, and this README.
3. Install and verify: `pnpm install && pnpm build && pnpm test`
4. Protect `main` (rulesets are not copied by templates):
   ```bash
   gh api --method POST repos/<owner>/<name>/rulesets --input .github/rulesets/protect-main.json
   gh repo edit --enable-merge-commit=false --delete-branch-on-merge
   ```
5. Keep or remove the sample `packages/ui`, and add apps under `apps/`.

## Adding a Next.js app

The `next` ESLint preset and the `nextjs` tsconfig preset are ready for an app under `apps/`:

```js
// apps/web/eslint.config.js
import next from "@repo/eslint-config/next";

export default next;
```

```json
// apps/web/tsconfig.json
{
  "extends": "@repo/typescript-config/nextjs.json",
  "include": [
    "next-env.d.ts",
    "**/*.ts",
    "**/*.tsx",
    ".next/types/**/*.ts",
    ".next/dev/types/**/*.ts"
  ],
  "exclude": ["node_modules"]
}
```

- Use `"typecheck": "next typegen && tsc --noEmit"` so route types exist before `tsc` runs on a clean checkout.
- The preset uses `@next/eslint-plugin-next` directly, not `eslint-config-next`: the plugins bundled with `eslint-config-next` do not support ESLint 10 yet.
- `.next/` is git-ignored and listed in the Turbo `build` outputs (without `.next/cache/`), so a second build is a cache hit.
- `@eslint-react/purity` also runs on Server Components, where reading `new Date()` during render is fine. Pass such values in as arguments, or turn the rule off for those files.

## Adding a Vite app

For a single-page app without a server, use the `vite` presets:

```js
// apps/web/eslint.config.js
import vite from "@repo/eslint-config/vite";

export default vite;
```

```json
// apps/web/tsconfig.json
{
  "extends": "@repo/typescript-config/vite-app.json",
  "include": ["src", "vite.config.ts"],
  "compilerOptions": { "types": ["vite/client", "node"] }
}
```

- The ESLint preset is the React preset plus Node globals for `*.config.*` files.
- Add `@types/node` to the app if `vite.config.ts` uses Node APIs (as above).

## Progress

The step-by-step build log, with the reasoning behind each decision and the issues met along the way, is in [PROGRESS.md](PROGRESS.md).

## License

[MIT](LICENSE) © 2026 Duc Tran
