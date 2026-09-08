# Discovery checklist

Per-ecosystem guide to the files that reveal a repo's stack, structure, and setup.
Read the section(s) matching what you find at the repo root. The goal of recon is to
*confirm* facts from real files, not infer them from directory names.

## Contents

- [Any repo — always check](#any-repo--always-check)
- [JavaScript / TypeScript](#javascript--typescript)
- [Python](#python)
- [Go](#go)
- [Rust](#rust)
- [Ruby](#ruby)
- [Java / Kotlin / JVM](#java--kotlin--jvm)
- [C# / .NET](#c--net)
- [PHP](#php)
- [Reading the layout](#reading-the-layout)

## Any repo — always check

| File | What it tells you |
| --- | --- |
| `README*` | Stated purpose, setup steps, run commands, architecture notes |
| `CONTRIBUTING*`, `docs/`, `ARCHITECTURE*`, `ADR*` / `adr/` | Conventions, design decisions, the maintainers' own mental model |
| `Makefile`, `Justfile`, `Taskfile.yml`, `package.json` scripts, `scripts/` | The real entry points people use day to day — build, test, run, lint, codegen |
| `.env.example`, `.env.sample`, `env.template` | Required configuration; diff against what the code actually reads |
| `Dockerfile`, `docker-compose.yml`, `devcontainer.json` | Runtime, services, ports, dependencies, how the app is packaged |
| `.github/workflows/`, `.gitlab-ci.yml`, `.circleci/`, `Jenkinsfile`, `azure-pipelines.yml` | Build/test/deploy pipeline, target environments, matrix versions, release process |
| `.tool-versions`, `.nvmrc`, `.python-version`, `mise.toml`, `rust-toolchain*` | Pinned toolchain versions |
| `.git/config`, remotes | Where it lives, monorepo vs single-purpose |
| Lockfiles | Exact dependency versions; presence tells you the package manager |
| Migration dirs (`migrations/`, `db/migrate/`, `alembic/`, `prisma/migrations/`) | Schema shape and history, whether migrations are a required setup step |
| `.editorconfig`, linter configs | Formatting/lint conventions enforced in CI |

## JavaScript / TypeScript

- **Manifest:** `package.json` — read `dependencies`, `devDependencies`, `scripts`, `engines`, `workspaces`, `type` (module vs commonjs).
- **Package manager:** `package-lock.json` → npm, `yarn.lock` → Yarn, `pnpm-lock.yaml` → pnpm, `bun.lockb` → Bun.
- **Monorepo:** `workspaces` in `package.json`, `pnpm-workspace.yaml`, `turbo.json`, `nx.json`, `lerna.json`, `packages/` or `apps/`.
- **Language/build:** `tsconfig.json` (check `paths`, `baseUrl`, `strict`), bundler config — `vite.config`, `webpack.config`, `next.config`, `rollup.config`, `esbuild`, `tsup.config`.
- **Framework signals:** `next`, `react`, `vue`, `@angular/core`, `svelte`, `@nestjs/core`, `express`, `fastify`, `koa`, `@remix-run`, `astro` in deps.
- **Tests:** `jest.config`, `vitest.config`, `playwright.config`, `cypress.config`, `*.test.ts` / `*.spec.ts` placement.
- **ORM/data:** `prisma/schema.prisma`, `drizzle.config`, `typeorm`, `sequelize`, `mongoose`, `knexfile`.
- **Entry points:** `main` / `module` / `exports` / `bin` in `package.json`; `src/index`, `src/main`, `src/server`, `app/`, `pages/`, `src/app/`.

## Python

- **Manifest:** `pyproject.toml` (PEP 621 `[project]`, or Poetry `[tool.poetry]`, or PDM/Hatch/uv sections), else `setup.py` / `setup.cfg`, else `requirements*.txt`.
- **Package manager / env:** `poetry.lock` → Poetry, `uv.lock` → uv, `Pipfile.lock` → Pipenv, `conda`/`environment.yml` → Conda, plain `requirements.txt` → pip.
- **Python version:** `[project] requires-python`, `.python-version`, classifiers, CI matrix.
- **Framework signals:** `django`, `flask`, `fastapi`, `starlette`, `celery`, `pydantic`, `sqlalchemy`, `scrapy` in deps. Django → also read `settings.py` / `settings/`, `INSTALLED_APPS`, `urls.py`, `manage.py`, `wsgi.py` / `asgi.py`.
- **Tests:** `pytest.ini` / `tox.ini` / `[tool.pytest]`, `conftest.py`, `tests/`. Note fixtures that need a DB or network.
- **Data:** `alembic/` + `alembic.ini`, Django `migrations/`, `models.py` / `models/`.
- **Entry points:** `[project.scripts]`, `__main__.py`, `manage.py`, `app.py` / `main.py`, ASGI/WSGI callables.

## Go

- **Manifest:** `go.mod` (module path, Go version, `require` block), `go.sum`.
- **Workspace:** `go.work` for multi-module.
- **Framework signals:** `gin`, `echo`, `chi`, `fiber`, `gorilla/mux`, `grpc`, `ent`, `gorm`, `sqlc`, `cobra` (CLI), `viper` (config).
- **Layout convention:** `cmd/<name>/main.go` for entry points, `internal/` for private packages, `pkg/` for exported libs, `api/` for protobuf/OpenAPI.
- **Tests:** `*_test.go` next to source; check for `//go:build integration` tags and `testdata/` dirs.
- **Codegen:** `//go:generate` directives, `sqlc.yaml`, `buf.yaml` / `.proto` files, `wire_gen.go`.

## Rust

- **Manifest:** `Cargo.toml` — `[dependencies]`, `[workspace]` members, `[[bin]]` / `[lib]`, edition, `rust-version`.
- **Lockfile:** `Cargo.lock`.
- **Framework signals:** `actix-web`, `axum`, `rocket`, `tokio`, `tonic` (gRPC), `diesel` / `sqlx` / `sea-orm`, `clap` (CLI), `serde`.
- **Layout:** `src/main.rs` / `src/lib.rs` / `src/bin/`, workspace crates in `crates/`.
- **Tests:** inline `#[cfg(test)]` modules, `tests/` for integration, `build.rs` for build scripts.

## Ruby

- **Manifest:** `Gemfile` + `Gemfile.lock`, `*.gemspec` for gems.
- **Ruby version:** `.ruby-version`, `Gemfile` `ruby` directive.
- **Rails signals:** `config/routes.rb`, `config/application.rb`, `app/{models,controllers,views,jobs,services}`, `db/schema.rb` / `db/migrate/`, `bin/rails`.
- **Tests:** `spec/` (RSpec) vs `test/` (Minitest), `.rspec`, factories in `spec/factories/`.
- **Background work:** `sidekiq`, `resque`, `good_job`, `config/sidekiq.yml`, `app/jobs/`.

## Java / Kotlin / JVM

- **Manifest:** `pom.xml` (Maven) or `build.gradle` / `build.gradle.kts` + `settings.gradle` (Gradle).
- **Multi-module:** `<modules>` in `pom.xml`, `include(...)` in `settings.gradle`.
- **Framework signals:** `spring-boot-starter-*`, `micronaut`, `quarkus`, `jakarta` / `javax`, `hibernate`, `jooq`, `mybatis`.
- **Layout:** `src/main/java` (or `kotlin`), `src/main/resources` (`application.yml` / `application.properties`), `src/test/java`.
- **Entry points:** class with `main` / `@SpringBootApplication`; check `application.yml` for ports, datasources, profiles.

## C# / .NET

- **Manifest:** `*.csproj` / `*.fsproj`, `*.sln`, `Directory.Build.props`, `Directory.Packages.props` (central package management).
- **Target framework:** `<TargetFramework>` (e.g. `net8.0`).
- **Framework signals:** `Microsoft.AspNetCore.*`, `Microsoft.EntityFrameworkCore.*`, `MediatR`, `Dapper`, `Serilog`.
- **Config:** `appsettings.json` / `appsettings.*.json`, `Program.cs` (and `Startup.cs` in older projects) — DI registrations, middleware pipeline, connection strings.
- **Layout:** project-per-concern (`*.Api`, `*.Domain`, `*.Infrastructure`, `*.Tests`); EF migrations in `Migrations/`.
- **Tests:** xUnit / NUnit / MSTest projects, usually `*.Tests` or `test/`.

## PHP

- **Manifest:** `composer.json` + `composer.lock` (`require`, `require-dev`, `autoload` PSR-4 map, `scripts`).
- **Framework signals:** `laravel/framework`, `symfony/*`, `slim/slim`, `cakephp`.
- **Laravel:** `routes/`, `app/{Http,Models,Jobs,Providers}`, `config/`, `database/migrations/`, `artisan`, `.env`.
- **Symfony:** `config/`, `src/Controller/`, `bin/console`, `symfony.lock`.
- **Tests:** `phpunit.xml`, `tests/`, Pest (`pest.php`).

## Reading the layout

Once you know the ecosystem, figure out the **organizing principle** — it decides where a new feature goes:

- **By layer / technical role:** top-level `controllers/`, `services/`, `models/`, `repositories/`. A feature spreads across all of them.
- **By feature / domain / module:** top-level `users/`, `billing/`, `orders/`, each containing its own controller/service/model. A feature is mostly one folder.
- **Hexagonal / clean / onion:** `domain/` (pure logic), `application/` (use cases), `infrastructure/` (DB, HTTP, external), `interfaces/` or `adapters/`. Dependencies point inward.
- **Framework-dictated:** Rails/Laravel/Django impose their own (`app/models`, `app/controllers`, …); the interesting question is where the team put logic that doesn't fit the framework's boxes — often `app/services/`, `lib/`, `domain/`.

Confirm by opening one real feature and following it across files. The "add X" recipes in the Repo layout section should mirror that real example.
