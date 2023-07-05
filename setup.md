---
layout: page
title: Set up GUAC with Docker Compose
permalink: /setup/
parent: Getting started with GUAC
nav_order: 1
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

{: .no_toc }

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
- [Git](https://git-scm.com/downloads)
- [Go](https://go.dev/doc/install)
- [GoReleaser](https://goreleaser.com/)
- [Make](https://www.gnu.org/software/make/)
- [jq](https://stedolan.github.io/jq/download/) (optional)

## Step 1: Clone GUAC

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

## Step 2: Build the containers

From your GUAC directory, run:

```bash
make container
```

## Step 3: Start the GUAC server

1. In another terminal, from your GUAC directory, run:

   ```bash
   docker compose up --force-recreate
   ```

2. Verify that GUAC is running:

   ```bash
   docker compose ls --filter "name=guac"
   ```

   You should see:

   ```bash
   NAME                STATUS              CONFIG FILES
   guac                running(7)          /Users/lumb/go/src/github.com/guacsec/guac/docker-compose.yml
   ```

   **If you don’t see the above,** run `docker-compose down` and try starting up
   GUAC again. Because Docker Compose caches the containers used, the unclean
   state can cause issues.

### GUAC Ports

| Port Number | GUAC Component | Note                                                                                                                                                                                                            |
| ----------- | -------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| 8080        | GraphQL server | To see the GraphQL playground, visit [http://localhost:8080](http://localhost:8080)                                                                                                                             |
| 4222        | Nats           | This is where any collectors that you run will need to connect to push any docs they find. The GUAC collector command defaults to `nats://127.0.0.1:4222` for the Nats address, so this will work automatically |

## Step 4: Start Ingesting Data

Build the GUAC binaries for your local machine and run them natively:

```bash
make build
```

You can run the `guacone collect files` ingestion command to load data into your
GUAC deployment. For example we can ingest the sample `guac-data` data. However,
you may ingest what you wish to here instead.

```bash
./bin/guacone collect files ../guac-data/docs
```

Switch back to the compose window and you will soon see that the OSV certifier
recognized the new packages and is looking up vulnerability information for
them.

## Step 5: Check that everything is ingesting and running

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
