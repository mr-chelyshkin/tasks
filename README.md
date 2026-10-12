# Containerized Taskfiles

[![License: Apache-2.0](https://img.shields.io/github/license/mr-chelyshkin/tasks?label=license)](LICENSE)

<p align="center">
  <img src=".github/assets/readme-header.png"
       alt="github.com/mr-chelyshkin/tasks"
       width="800">
</p>

### Define a project command once. Run it locally and in CI with containerized tools.

Reusable [Task](https://taskfile.dev/) modules that run development tools in
containers. Developers and CI call the same project command in the same image.
See the [concept](concept.md).

## Quick start

Requires [Task 3.53.1+](https://taskfile.dev/docs/installation), Git and Docker.

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

```sh
task ci/build                                           # locally
task --trusted-hosts raw.githubusercontent.com ci/build # in CI
```

## Modules

| Module | Tools |
| -- | -- |
| [Go](taskfiles/golang/Taskfile.yml) | gofumpt, golangci-lint, govulncheck |
| [Rust](taskfiles/rust/Taskfile.yml) | cargo, clippy, cargo-audit |
| [Python](taskfiles/python/Taskfile.yml) | uv, Ruff, mypy, pytest |
| [Node.js](taskfiles/node/Taskfile.yml) | npm |
| [Nix](taskfiles/nix/Taskfile.yml) | nixfmt, statix, deadnix |
| [Protobuf](taskfiles/proto/Taskfile.yml) | Buf, clang-format |
| [Terraform](taskfiles/terraform/Taskfile.yml) | Terraform |
| [Markdown](taskfiles/markdown/Taskfile.yml) | mdformat |
| [AWS](taskfiles/aws/Taskfile.yml) | AWS CLI |

`task --list` shows the tasks of every non-internal include.

## Conventions

- `<verb>` checks, `<verb>/fix` writes: `fmt`, `lint`, `test`, `audit`,
  `typecheck`, `docs`, `build`. Checks mount the project read-only.
- `<TASK>_ARGS` passes extra arguments to a task: `FMT_ARGS`, `TEST_ARGS`.
- The image comes from `<AREA>_IMAGE`, `<AREA>_VERSION`, the project's pin file
  (`go.mod`, `rust-toolchain.toml`, `.python-version`, …) or the module default.
- `CONTAINER_*` options and `PROJECT_ROOT` work in every module; credentials are
  forwarded with `CONTAINER_ENVS` and `CONTAINER_VOLUMES`.
- `<module>:exec CMD=…` runs any command in the module's container.

## Related repositories

- [Reusable GitHub Actions](https://github.com/mr-chelyshkin/actions)
- [Container images](https://github.com/mr-chelyshkin/images)
