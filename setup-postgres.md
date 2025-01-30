---
layout: page
title: Start GUAC with PostgreSQL using Docker Compose
permalink: /setup-postgres/
parent: Getting started with GUAC
nav_order: 4
---

# Start GUAC with PostgreSQL using Docker Compose

{: .note }

If you’d prefer, you can set up GUAC with Kubernetes with the experimental
[Helm charts provided by Kusari](https://github.com/kusaridev/helm-charts/tree/main/charts/guac).
Note that these helm charts are still experimental and are hosted in a
third-party repo and may not be synchronized with the GUAC repo.

This tutorial will walk you through how to deploy a full persistant GUAC
deployment with a PostgreSQL database backend using Docker Compose.

## Prerequisites

- [Docker](https://docs.docker.com/get-docker/)
- [Docker Compose](https://docs.docker.com/compose/install/)
- [jq](https://stedolan.github.io/jq/download/)

## Optional - Verify images and binaries

- Follow [Verification of the GUAC images and
  binaries]({{ site.baseurl }}{%link verification.md %})

## Step 1: Download GUAC

1. Download the GUAC CLI `guaccollect` binary for your machine's OS and
   architecture from the
   [latest GUAC release](https://github.com/guacsec/guac/releases/latest) if you
   have not already done so. For example:

   - Linux x86_64 :
     [`guaccollect-linux-amd64`](https://github.com/guacsec/guac/releases/latest/download/guaccollect-linux-amd64)
   - MacOS x86_64 :
     [`guaccollect-darwin-amd64`](https://github.com/guacsec/guac/releases/latest/download/guaccollect-darwin-amd64)
   - Windows x86_64 :
     [`guaccollect-windows-amd64.exe`](https://github.com/guacsec/guac/releases/latest/download/guaccollect-windows-amd64.exe)

1. Rename the binary to `guaccollect`, mark it executable if necessary, and add
   it to your shell's path.

1. Download the
   [compose yaml](https://github.com/guacsec/guac/releases/latest/download/guac-postgres-compose.yaml)
   from the
   [latest GUAC release](https://github.com/guacsec/guac/releases/latest).

1. Optional: If you want test data to use,
   [download and unzip GUAC’s test data.](https://github.com/guacsec/guac-data/archive/refs/heads/main.zip)

## Step 2: Start the GUAC server

1. From the directory you downloaded the `guac-postgres-compose.yaml`, run:

   ```bash
   docker compose -f guac-postgres-compose.yaml up
   ```

1. Verify that GUAC is running:

   ```bash
   docker compose ls
   ```

   You should see:

   ```bash
   NAME                STATUS              CONFIG FILES
   dirname             running(9)          /files/dirname/guac-postgres-compose.yml
   ```

   **If you don’t see the above,** run `docker compose down` and try starting up
   GUAC again. Because Docker Compose caches the containers used, the unclean
   state can cause issues.

### GUAC Ports

| Port Number | GUAC Component       | Note                                                                                                                                                     |
| ----------- | -------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------- |
| 8080        | GraphQL server       | To see the GraphQL playground, visit [http://localhost:8080](http://localhost:8080).                                                                     |
| 2782        | Collector Subscriber | This service is notified whenever you run a collector, such as `guacone collect files`. Then subscribers can collect more data on any packages ingested. |
| 4222        | Nats                 | Ingestion pubsub endpoint                                                                                                                                |
| 8081        | REST server          | GUAC endpoint for simplified REST queries.                                                                                                               |

### GUAC Volume Mounts

Two directories are created in the same directory as the compose file, these are
used for:

- **blobstore**: This directory is a temporary storage of documents that are
  being queued to the ingestor.

- **postgres-data**: This directory contains the postgres database files.

## Step 3: Start Ingesting Data

Before ingesting data, the blobstore directory must be writable by your local
user. Because it was created by a the Ingestor docker container, it will have a
different user id.

```bash
sudo chmod a+w ./blobstore
```

You can run the `guaccollect files` ingestion command to load data into your
GUAC deployment. For example we can ingest the sample `guac-data` data. However,
you may ingest what you wish to here instead.

```bash
guaccollect files --service-poll=false --blob-addr=file://./blobstore?no_tmp_dir=true ./guac-data-main/docs
```

This command will take all documents under the `./guac-data-main/docs` directory
and ingest them into GUAC by placing messages on the Nats pubsub queue, and also
placing the documents in the `./blobstore` directory for the ingestor to pick
up.

Switch back to the compose window and you will soon see that the Ingestor is
peforming the parsing and GraphQL mutations to add the documents to GUAC. Also,
the deps.dev collector and OSV certifier have recognized the new packages and
are looking up dependency and vulnerability information for them.

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

- **PostgreSQL**: Serves as the persistant data store for all the GUAC data.

- **GraphQL Server**: Serves GUAC GraphQL queries and stores the data. As the
  in-memory backend is used, no separate backend is needed behind the server.

- **Collector-Subscriber**: Helps communicate to the collectors when additional
  information is needed.

- **Deps.dev Collector**: Gathers further information from
  [Deps.dev](https://deps.dev/) for supported packages.

- **OSV Certifier**: Gathers OSV vulnerability information from
  [osv.dev](https://osv.dev/) about packages.

- **Ingestor**: Retrieves ingestion messages from Nats, parses the documents,
  and mutates the GraphQL graph to add the data to GUAC.

- **Nats**: Serves as the pubsub to accept ingestion requests which the Ingestor
  pulls from.

- **OCI Collector**: Collects additional metadata from OCI registries about any
  container references found in documents.

- **GUAC REST Server**: Serves simplified query endpoints.

## Next steps

This compose configuration is suitable to leave running in an environment that
is accessible to your environment for further GUAC ingestion, discovery,
analysis, and evaluation. Explore the types of collectors available under the
`guaccollect collect` command and see what will work for your build, ingestion,
and SBOM workflow. These collectors can be run as another service that watches a
location for new documents to ingest. If you’re curious about the various GUAC
components and what they do, see [How GUAC components work together]({{
site.baseurl }}{%link guac-components.md %}).

You may wish to alter the volume configuration to change the blobstore and
postgres-data locations. The blobstore needs to be accessable to any
`guaccollect` commands and supports cloud buckets. The `guacone` command does
not need access to the blobstore and interacts directly with the GraphQL server.
