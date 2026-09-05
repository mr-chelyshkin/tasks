# Containerized Taskfiles

[![License: Apache-2.0](https://img.shields.io/github/license/mr-chelyshkin/images?label=license)](LICENSE)

<p align="center">
  <img src=".github/assets/readme-header.png"
       alt="An engraved crate-packing line with mechanical presses and a conveyor"
       width="800">
</p>

**Define a project command once. Run it locally and in CI with containerized tools.**

Local and CI workflows drift. Different tool versions, flags, and working directories turn one build into two systems to maintain.

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
    vars:
      NODE_IMAGE: docker.io/library/node:22.23.1-bookworm-slim
      APP_DIR:    .

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
| [Node.js](taskfiles/node/Taskfile.yml)        | Containerized npm workflows for Node.js projects.                    |
| [Rust](taskfiles/rust/Taskfile.yml)           | Containerized development and CI workflows for Rust projects.        |
| [Protobuf](taskfiles/proto/Taskfile.yml)      | Containerized validation and formatting for Protobuf schemas.        |
| [Terraform](taskfiles/terraform/Taskfile.yml) | Containerized workflows for Terraform configurations.                |
| [AWS](taskfiles/aws/Taskfile.yml)             | Containerized static-site delivery through Amazon S3 and CloudFront. |
| [Docker](taskfiles/docker/Taskfile.yml)       | Shared container runner used by the tool modules.                    |

## Configure and extend

- Set module defaults in `includes.<module>.vars`.
- Pass call-specific values in the task call's `vars` block.

Both are shown in the [Quick Start](#quick-start). The linked [Taskfiles](#available-modules) are the source of truth for module inputs and behavior. 
Container execution is defined by [`docker:run`](taskfiles/docker/Taskfile.yml).

For a command without a named task, call the module wrapper with `CMD`, for example [`node:_npm/tool`](taskfiles/node/Taskfile.yml).

Use `task --list-all` to discover callable tasks and `task --dry <task>` to inspect a command.

## Learn more

- [Concept and design boundaries](concept.md)
- [Taskfile composition and variables](https://taskfile.dev/docs/guide)
- [Taskfile schema versions](https://taskfile.dev/docs/taskfile-versions)
