# Containerized Taskfiles

[![License: Apache-2.0](https://img.shields.io/github/license/mr-chelyshkin/tasks?label=license)](LICENSE)

<p align="center">
  <img src=".github/assets/readme-header.png"
       alt="github.com/mr-chelyshkin/tasks"
       width="800">
</p>

### Define a project command once. Run it locally and in CI with containerized tools.

Local and CI workflows drift. 
Different tool versions, flags, and working directories turn one build into two systems to maintain.

This repository provides reusable [Task](https://taskfile.dev/) modules for running development and delivery tools in containers.
Include the modules your project needs, choose the tool images, and compose your project commands in a Taskfile.
Developers and CI then call those same commands.

- **Less host setup:** run tools without installing each language toolchain and CLI locally.
- **One execution path:** local development and CI call the same project task, shared module, and selected image.
- **Reuse across projects:** include versioned modules instead of copying shell scripts and `docker run` commands.
- **Adopt one task at a time:** keep your editor, application configuration, and existing CI provider.

Read the [concept](concept.md) for the reasoning and responsibility boundaries.

## Quick start

Requirements: [Task 3.53.1+](https://taskfile.dev/docs/installation), Git, and Docker on Linux or macOS.

In a npm project with a committed `package-lock.json` and a `build` script, create `Taskfile.yml`:

```yaml
version: '3.53.1'

includes:
  node:
    taskfile: https://raw.githubusercontent.com/mr-chelyshkin/tasks/main/taskfiles/node/Taskfile.yml

tasks:
  ci/build:
    desc: Build the application.
    cmds:
      - task: node:run
        vars:
          SCRIPT: build
```

Run it from your local environment:

```sh
task ci/build
```

`npm ci` and `npm run build` run in the selected container;
project files stay on the host and npm downloads are cached in `.cache/node`.

## Run the same task in CI

```sh
task --trusted-hosts raw.githubusercontent.com ci/build
```

The CI job owns checkout, credentials, scheduling, and artifact upload.
The Taskfile owns the build command.

## Available modules

Include only the modules your project needs. Follow the links for the exact tasks, inputs, and defaults.

| Module                                        | Purpose                                                              |
|-----------------------------------------------|----------------------------------------------------------------------|
| [Go](taskfiles/golang/Taskfile.yml)           | Containerized development and CI workflows for Go projects.          |
| [Markdown](taskfiles/markdown/Taskfile.yml)   | Containerized Markdown formatting with configurable style.           |
| [Nix](taskfiles/nix/Taskfile.yml)             | Containerized formatting, linting, and Nix commands.                 |
| [Node.js](taskfiles/node/Taskfile.yml)        | Containerized npm workflows for Node.js projects.                    |
| [Python](taskfiles/python/Taskfile.yml)       | Containerized uv workflows for Python projects.                      |
| [Rust](taskfiles/rust/Taskfile.yml)           | Containerized development and CI workflows for Rust projects.        |
| [Protobuf](taskfiles/proto/Taskfile.yml)      | Containerized validation and formatting for Protobuf schemas.        |
| [Terraform](taskfiles/terraform/Taskfile.yml) | Containerized workflows for Terraform configurations.                |
| [AWS](taskfiles/aws/Taskfile.yml)             | Containerized static-site delivery through Amazon S3 and CloudFront. |
| [Docker](taskfiles/docker/Taskfile.yml)       | Shared container runner used by the tool modules.                    |

## Module contract

Every module follows the same rules, so a project that knows one module knows them all.

### Tasks

- `<verb>` checks and never changes files; `<verb>/fix` applies the changes with the same tool.
- Shared verbs: `fmt`, `lint`, `test`, `audit`, `typecheck`, `docs`, `build`. Other tasks keep their tool's vocabulary.
- `exec` runs any command in the module's container, with the module's mounts, cache and environment.

| Module    | Tasks                                                                                                                      |
|-----------|----------------------------------------------------------------------------------------------------------------------------|
| Go        | `exec` `fmt` `fmt/fix` `lint` `lint/fix` `test` `audit` `build` `tidy` `vendor`                                            |
| Rust      | `exec` `fmt` `fmt/fix` `lint` `lint/fix` `test` `audit` `audit/fix` `docs` `build` `check` `bench` `docs/nightly` `package/list` `publish/dry-run` |
| Python    | `exec` `fmt` `fmt/fix` `lint` `lint/fix` `test` `audit` `typecheck` `build` `package/check` `tag/check`                    |
| Node.js   | `exec` `audit` `run`                                                                                                       |
| Nix       | `exec` `fmt` `fmt/fix` `lint` `lint/fix`                                                                                   |
| Protobuf  | `exec` `fmt` `fmt/fix` `lint` `build` `breaking`                                                                           |
| Terraform | `exec` `fmt` `fmt/fix` `validate` `plan`                                                                                   |
| Markdown  | `exec` `fmt` `fmt/fix`                                                                                                     |
| AWS       | `exec` `s3/upload/static` `cloudfront/cache/invalidate`                                                                    |

`task --list` shows every task with its description.

### Variables

| Variable                         | Meaning                                                                                                                           |
|----------------------------------|-----------------------------------------------------------------------------------------------------------------------------------|
| `<TASK>_ARGS`                    | Arguments appended after the mode flags the task owns. Named after the task: `FMT_ARGS`, `DOCS_NIGHTLY_ARGS`. `<verb>/fix` reads the variable of `<verb>`; a task that runs several tools reads `<TOOL>_ARGS` (`STATIX_ARGS`, `DEADNIX_ARGS`). |
| `<AREA>_DIR`                     | Working directory, relative to `PROJECT_ROOT`. Default: `.`.                                                                      |
| `<AREA>_VERSION`, `<AREA>_IMAGE` | Tool image, see [Image version](#image-version).                                                                                  |
| `<AREA>_CACHE_DIR`               | Host cache directory, relative to `PROJECT_ROOT` (Go, Rust, Python, Node.js, Terraform). Default: `.cache/<area>`.                |
| `PROJECT_ROOT`                   | Directory mounted at `/workspace`. Default: the Git repository root.                                                              |
| `CONTAINER_*`                    | Container options shared by every module: `ENGINE`, `PULL_POLICY`, `MOUNT_MODE`, `USER`, `ENVS`, `VOLUMES`, `PORTS`, `RUN_ARGS`.  |

Area prefixes: `GO`, `RUST`, `PYTHON`, `NODE`, `NIX`, `PROTO`, `TERRAFORM`, `MARKDOWN`, `AWS`.
An empty value counts as unset.

### Image version

A module selects its image in this order:

1. `<AREA>_IMAGE`: any image reference.
2. `<AREA>_VERSION`: a tag of `ghcr.io/mr-chelyshkin/ci/<area>`.
3. The ecosystem's pin file in `<AREA>_DIR`.
4. The module default.

Image tags are exact versions, so a version must have the form `X.Y.Z`.

| Module                       | Pin file                                             | Default                                |
|------------------------------|------------------------------------------------------|----------------------------------------|
| Go                           | `go.mod`: `toolchain goX.Y.Z`, otherwise `go X.Y.Z` | —                                      |
| Rust                         | `rust-toolchain.toml`: `channel`                     | —                                      |
| Python                       | `.python-version`                                    | —                                      |
| Node.js                      | `.node-version`, then `.nvmrc`                       | `22.23.1`                              |
| Terraform                    | `.terraform-version`                                 | `1.15.9`                               |
| Nix, Protobuf, Markdown, AWS | —                                                    | `2.35.2`, `1.50.0`, `1.0.0`, `2.36.24` |

### Project-specific commands

`exec` takes `CMD` and splits it like a shell command line. Wrap shell snippets in single quotes:

```yaml
tasks:
  ci/generate:
    desc: Check that generated code is up to date.
    cmds:
      - task: golang:exec
        vars:
          CMD: sh -c 'go generate ./... && git diff --exit-code'
```

To change container options for one call, pass `TASK_CONTEXT`. Later maps win in `merge`, so
`merge (dict …) (.TASK_CONTEXT | default .)` sets a default and `merge (.TASK_CONTEXT | default .) (dict …)` forces a value:

```yaml
      - task: markdown:fmt
        vars:
          TASK_CONTEXT:
            ref: 'merge (dict "CONTAINER_MOUNT_MODE" "ro") (.TASK_CONTEXT | default .)'
```

### Upgrading from v0.0.x

| Before                                                                                       | After                                                               |
|----------------------------------------------------------------------------------------------|---------------------------------------------------------------------|
| `_go/tool`, `_cargo/tool`, `_uv/tool`, `_npm/tool`, `_nix/tool`, `_proto/tool`, `_terraform/tool`, `_markdown/tool`, `_aws/tool` | `exec` |
| `golang:gofumpt`, `golang:golangci`, `golang:govulncheck`                                    | `golang:fmt`, `golang:lint`, `golang:audit`                         |
| `rust:clippy`, `rust:doc/build`, `rust:doc/build-nightly`                                    | `rust:lint`, `rust:docs`, `rust:docs/nightly`                       |
| `nix:nixfmt`, `nix:nixfmt FMT_ARGS=''`, `nix:statix`, `nix:deadnix`                          | `nix:fmt`, `nix:fmt/fix`, `nix:lint`                                |
| `proto:format/clang`, `proto:format/clang/fix`                                               | `proto:fmt`, `proto:fmt/fix`                                        |
| `VULN_ARGS`, `DOC_ARGS`, `FORMAT_ARGS`, `FMT_PATHS`                                          | `AUDIT_ARGS`, `DOCS_ARGS`, `FMT_ARGS`, `FMT_ARGS`                   |
| `BUF_ARGS`                                                                                   | `LINT_ARGS` for `lint`, `BUILD_ARGS` for `build`                    |
| `PACKAGE_ARGS`, `PUBLISH_ARGS`                                                               | `PACKAGE_LIST_ARGS`, `PUBLISH_DRY_RUN_ARGS`                         |
| `APP_DIR` (default `app`), `AUDIT_LEVEL`                                                     | `NODE_DIR` (default `.`), `AUDIT_ARGS` (default `--audit-level=high`) |
| `TERRAFORM_DIR` default `tf`                                                                 | default `.`                                                         |
| Mode flags in `*_ARGS`: `-l` (gofumpt), `--check` (cargo fmt, ruff), `-check -diff` (terraform) | Owned by the task; pass only extra arguments                     |
| `STATIX_ARGS` / `DEADNIX_ARGS` with targets and `--fail`; nixfmt over every `*.nix` file      | Options only; Nix and Protobuf tasks take files from Git, honouring `.gitignore` |
| Rust version from `rust-version` in `Cargo.toml`                                             | `channel` in `rust-toolchain.toml`, or `RUST_VERSION`               |

## Learn more

- [Concept and design boundaries](concept.md)
- [Taskfile composition and variables](https://taskfile.dev/docs/guide)
- [Taskfile schema versions](https://taskfile.dev/docs/taskfile-versions)
