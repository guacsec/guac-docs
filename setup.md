---
layout: page
title: Set up GUAC with Docker Compose
permalink: /setup/
parent: Getting started with GUAC
nav_order: 2
---

# Set up GUAC with Docker Compose

{: .note }

If you’d prefer, you can set up GUAC with Kubernetes with the experimental
[Helm charts provided by Kusari](https://github.com/kusaridev/helm-charts/tree/main/charts/guac).
Note that these helm charts are still experimental and are hosted in a
third-party repo and may not be synchronized with the GUAC repo.

GUAC consists of multiple components. You may have seen a subset of these used
in various [GUAC demos]({{ site.baseurl }}{% link guac-use-cases.md %}). To get
the most value out of GUAC, you’ll need to set up all components. This tutorial
will walk you through how to deploy GUAC, using Docker Compose, so that you get
the full set of components.

If you’re curious about the various GUAC components and what they do, see [How
GUAC components work together]({{ site.baseurl }}{%link guac-components.md %}).

## Setup video

A video format of these setup instructions is available here:

<iframe width="560" height="315" src="https://www.youtube-nocookie.com/embed/3e-Qurgl3Sc?si=N2z7AAUOj1lM1EG-" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" allowfullscreen></iframe>

<details open markdown="block">
  <summary>
    Table of contents
  </summary>
  {: .text-delta }
- TOC
{:toc}
</details>

## Prerequisites

- [Docker](https://docs.docker.com/get-docker/)
- [Docker Compose](https://docs.docker.com/compose/install/)
- [jq](https://stedolan.github.io/jq/download/)

## Optional - Verify images and binaries

- Follow [Verification of the GUAC images and
  binaries]({{ site.baseurl }}{%link verification.md %})

## Step 1: Download GUAC

1. Download the GUAC CLI `guacone` binary for your machine's OS and architecture
   from the
   [latest GUAC release](https://github.com/guacsec/guac/releases/latest) if you
   have not already done so. For example Linux x86_64 is
   [`guacone-linux-amd64'](https://github.com/guacsec/guac/releases/latest/download/guacone-linux-amd64).

1. Rename the binary to `guacone`, mark it executable if necessary, and add it
   to your shell's path.

1. Download the compose files from the
   [latest GUAC release](https://github.com/guacsec/guac/releases/latest). Look
   for an attached file `guac-compose-<release-tag>.tar.gz`. At the time of
   writing, this is:
   [`guac-compose-v0.1.1.tar.gz`](https://github.com/guacsec/guac/releases/download/v0.1.1/guac-compose-v0.1.1.tar.gz).

1. Untar the compose files and change to that directory. (the rest of the steps
   need to be done from this directory):

   ```bash
   tar zxvf guac-compose-v0.1.1.tar.gz
   cd guac-compose
   ```

1. Optional: If you want test data to use,
   [download and unzip GUAC’s test data.](https://github.com/guacsec/guac-data/archive/refs/heads/main.zip)

## Step 2: Start the GUAC server

1. In another terminal, from the `guac-compose` directory, run:

   ```bash
   docker compose up --force-recreate
   ```

1. Verify that GUAC is running:

   ```bash
   docker compose ls --filter "name=guac"
   ```

   You should see:

   ```bash
   NAME                STATUS              CONFIG FILES
   guac                running(7)          /Users/lumb/go/src/github.com/guacsec/guac/docker-compose.yml
   ```

   **If you don’t see the above,** run `docker compose down` and try starting up
   GUAC again. Because Docker Compose caches the containers used, the unclean
   state can cause issues.

### GUAC Ports

| Port Number | GUAC Component | Note                                                                                                                                                                                                            |
| ----------- | -------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| 8080        | GraphQL server | To see the GraphQL playground, visit [http://localhost:8080](http://localhost:8080)                                                                                                                             |
| 4222        | Nats           | This is where any collectors that you run will need to connect to push any docs they find. The GUAC collector command defaults to `nats://127.0.0.1:4222` for the Nats address, so this will work automatically |

## Step 3: Start Ingesting Data

You can run the `guacone collect files` ingestion command to load data into your
GUAC deployment. For example we can ingest the sample `guac-data` data. However,
you may ingest what you wish to here instead.

```bash
guacone collect files guac-data-main/docs
```

Switch back to the compose window and you will soon see that the OSV certifier
recognized the new packages and is looking up vulnerability information for
them.

## Step 4: Check that everything is ingesting and running

Run:

```bash
curl 'http://localhost:8080/query' -s -X POST -H 'content-type: application/json' \
  --data '{
    "query": "{ packages(pkgSpec: {}) { type } }"
  }' | jq
```

You should see the types of all the packages ingested

```json
{
  "data": {
    "packages": [
      {
        "type": "oci"
      },
...
```

## What is running?

Congratulations, you are now running a full GUAC deployment! Taking a look at
the `docker-compose.yaml` we can see what is actually running:

- **Nats**: Used for communication between the GUAC components. It is available
  on port `4222`.
- **Collector-Subscriber**: Helps communicate to the collectors when additional
  information is needed.
- **GraphQL Server**: Serves GUAC GraphQL queries and stores the data. As the
  in-memory backend is used, no separate backend is needed behind the server.
- **Ingestor**: Listens for things to ingest through Nats, then pushes to the
  GraphQL Server. The ingestor also runs the assembler and parser internally.
- **Image Collector**: Can pull OCI image metadata (SBOMs and attestations) from
  registries for further inspection.
- **Deps.dev Collector**: Gathers further information from
  [Deps.dev](https://deps.dev/) for supported packages.
- **OSV Certifier**: Gathers OSV vulnerability information from
  [osv.dev](https://osv.dev/) about packages.

## Next steps

The compose configuration is suitable to leave running in an environment that is
accessible to your environment for further GUAC ingestion, discovery, analysis,
and evaluation. Keep in mind that the in-memory backend is not persistent.
Explore the types of collectors available in the `collector` binary and see what
will work for your build, ingestion, and SBOM workflow. These collectors can be
run as another service that watches a location for new documents to ingest.
