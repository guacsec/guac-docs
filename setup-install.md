---
layout: page
title: Install and start GUAC
permalink: /setup-install/
parent: Start a demo GUAC with Docker Compose
nav_order: 1
---

# Install and start GUAC

## Prerequisites

- [Docker](https://docs.docker.com/get-docker/)
- [Docker Compose](https://docs.docker.com/compose/install/)
- [jq](https://stedolan.github.io/jq/download/)

## Optional - Verify images and binaries

- Follow [Verification of the GUAC images and
  binaries]({{ site.baseurl }}{%link verification.md %})

## Download GUAC

1. Download the GUAC CLI `guacone` binary for your machine's OS and architecture
   from the
   [latest GUAC release](https://github.com/guacsec/guac/releases/latest) if you
   have not already done so. For example:

   - Linux x86_64 :
     [`guacone-linux-amd64`](https://github.com/guacsec/guac/releases/latest/download/guacone-linux-amd64)
   - MacOS ARM64 :
     [`guacone-darwin-arm64`](https://github.com/guacsec/guac/releases/latest/download/guacone-darwin-arm64)
   - Windows x86_64 :
     [`guacone-windows-amd64.exe`](https://github.com/guacsec/guac/releases/latest/download/guacone-windows-amd64.exe)

1. Rename the binary to `guacone`, mark it executable if necessary, and add it
   to your shell's path.

1. Download the
   [compose yaml](https://github.com/guacsec/guac/releases/latest/download/guac-demo-compose.yaml)
   from the
   [latest GUAC release](https://github.com/guacsec/guac/releases/latest).

1. Download and unzip
   [the demo data](https://github.com/guacsec/guac-data/archive/refs/heads/main.zip)
   used in these examples.

## Start the GUAC server

1. From the directory you downloaded the `guac-demo-compose.yaml`, run:

   ```bash
   docker compose -f guac-demo-compose.yaml -p guac up --force-recreate
   ```

1. Verify that GUAC is running:

   ```bash
   docker compose ls --filter "name=guac"
   ```

   You should see:

   ```bash
   NAME                STATUS              CONFIG FILES
   guac                running(2)          /Users/lumb/go/src/github.com/guacsec/guac/docker-compose.yml,/Users/lumb/go/src/github.com/guacsec/guac/guac/container_files/mem.yaml
   guac-demo           running(5)          /Users/lumb/go/src/github.com/guacsec/guac-demo/guac-demo-compose.yaml
   ```

   **If you don’t see the above,** run `docker compose down` and try starting up
   GUAC again. Because Docker Compose caches the containers used, the unclean
   state can cause issues.

### GUAC Ports

| Port Number | GUAC Component       | Note                                                                                                                                                     |
| ----------- | -------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------- |
| 8080        | GraphQL server       | To see the GraphQL playground, visit [http://localhost:8080](http://localhost:8080)                                                                      |
| 2782        | Collector Subscriber | This service is notified whenever you run a collector, such as `guacone collect files`. Then subscribers can collect more data on any packages ingested. |

Now that you've installed GUAC, it's time to [ingest
data]({{ site.baseurl }}{% link setup-ingest-data.md %}).
