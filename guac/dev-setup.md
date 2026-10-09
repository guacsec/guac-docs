---
layout: page
title: Setting up your development environment for GUAC
permalink: /guac/dev-setup/
redirect_from: /dev-setup/
parent: Getting started with GUAC
nav_order: 6
---

# Development environment setup

## Try GUAC with a dev container

A [development container](https://containers.dev/) is a quick way to try GUAC
without installing its Go build tools locally. The GUAC repository includes a
[`.devcontainer/devcontainer.json`](https://github.com/guacsec/guac/blob/main/.devcontainer/devcontainer.json)
configuration that downloads a published GUAC image and starts the services
using Docker Compose.

Choose either of these environments:

- **GitHub Codespaces:** Open the
  [GUAC repository](https://github.com/guacsec/guac), select **Code**,
  **Codespaces**, then **Create codespace on main**.
- **VS Code locally:** Install
  [Docker](https://docs.docker.com/get-docker/) and the
  [Dev Containers extension](https://marketplace.visualstudio.com/items?itemName=ms-vscode-remote.remote-containers).
  Clone `https://github.com/guacsec/guac.git`, open the repository in VS Code,
  and select **Dev Containers: Reopen in Container** from the Command Palette.

The first startup pulls the GUAC release image. On each container start, its
`postStartCommand` launches the in-memory backend using
`docker-compose.yml` and `container_files/mem.yaml`. Wait for the terminal
message `GUAC is ready.`

Open the forwarded **8080** port to access the GraphQL playground. The REST API
is exposed on port **8081**, and you can check its health at
`http://localhost:8081/healthz` from inside the dev container (or through the
forwarded port).

From a terminal in the GUAC repository, you can manage the services:

```bash
# Show service status
docker compose -f docker-compose.yml -f container_files/mem.yaml ps

# Follow logs when debugging startup
docker compose -f docker-compose.yml -f container_files/mem.yaml logs -f

# Stop the services when you are done
docker compose -f docker-compose.yml -f container_files/mem.yaml down
```

The default backend is in memory, so it is intended for experimentation, not
durable storage. For a PostgreSQL-backed setup, replace
`container_files/mem.yaml` with `container_files/ent.yaml` in the
devcontainer's `postStartCommand` before reopening the container. See the
[GraphQL guide]({{ site.baseurl }}{% link guac/guac-graphql.md %}) for query and
ingestion examples.

## Install tools for manual development

## Ensure you have the following tools installed in your environment

- [Docker](https://docs.docker.com/get-docker/)
- [Docker Compose](https://docs.docker.com/compose/install/)
- [Docker Buildx](https://docs.docker.com/build/concepts/overview/#buildx)
- [Git](https://git-scm.com/downloads)
- [Go](https://go.dev/doc/install) (v1.26+)
- [GoReleaser](https://goreleaser.com/)
- [Make](https://www.gnu.org/software/make/)
- [jq](https://stedolan.github.io/jq/download/)
- [protoc](https://grpc.io/docs/protoc-installation/)
- [golangci-lint](https://golangci-lint.run/welcome/install/)
- [mockgen](https://github.com/uber-go/mock)
- [atlas](https://atlasgo.io/getting-started#installation)

Once you have cloned the repository, `make check-tools` verifies that the
required tools are present and tells you which are missing.

# Setting up your git repositories

1. Clone GUAC to a local directory:

   ```bash
   git clone https://github.com/guacsec/guac.git
   ```

2. Optional: If you want test data to use, clone GUAC’s test data:

   ```bash
   git clone https://github.com/guacsec/guac-data.git
   ```

3. Go to your GUAC directory (the rest of the steps need to be done from this
   directory):

   ```bash
   cd guac
   ```

# Building the binaries

All steps assume you are in the root of the guac directory.

1. Build the binaries using make:

   ```bash
   make
   ```

1. Alternatively, you may also build/run the binaries directly with go:

   ```bash
   go run ./cmd/guacgql --gql-debug
   ```

# Building containers

All steps assume you are in the root of the guac directory.

1. Build the binaries using make:

   ```bash
   make container
   ```

# Making changes to GraphQL (optional)

1. When making changes to the graphQL
   [schema](https://github.com/guacsec/guac/tree/main/pkg/assembler/graphql/schema)
   and
   [client operations](https://github.com/guacsec/guac/tree/main/pkg/assembler/clients/operations),
   you will need to run the graphQL generation code:

   ```bash
   make generate
   ```

# Making changes to protos (optional)

1. When making changes to any protocol buffers (e.g. for the [collectsub
   service]), you will need to run code generation:

   ```bash
   make proto
   ```

# Creating a PR

Whenever you are ready to contribute, feel free to open a pull request to the
repository! Whenever you are updating your branch, please be sure to
[rebase instead of creating a merge commit](https://www.geeksforgeeks.org/rebasing-of-branches-in-git/#).

# Next steps

Check out more information about becoming a contributor in the
[contributor guide](https://github.com/guacsec/guac/blob/main/CONTRIBUTING.md).
