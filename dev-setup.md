---
layout: page
title: Setting up your development environment for GUAC
permalink: /dev-setup/
parent: Getting started with GUAC
nav_order: 6
---

# Development environment setup

## Ensure you have the following tools installed in your environment

- [Docker](https://docs.docker.com/get-docker/)
- [Docker Compose](https://docs.docker.com/compose/install/)
- [Git](https://git-scm.com/downloads)
- [Go](https://go.dev/doc/install) (v1.21+)
- [GoReleaser](https://goreleaser.com/)
- [Make](https://www.gnu.org/software/make/)
- [jq](https://stedolan.github.io/jq/download/)
- [protoc](https://grpc.io/docs/protoc-installation/)

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
