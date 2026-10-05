---
layout: page
title: GUAC Configuration Guide
permalink: /guac/guac-configuration/
redirect_from: /guac-configuration/
parent: Getting started with GUAC
nav_order: 7
---

# GUAC Configuration Guide

This document provides an overview of the `guac.yaml` configuration file used in
GUAC deployments. It describes each configuration option, its default value, and
scenarios where you might want to change it.

{: .note }

The values below are the ones shipped in `guac.yaml`, which is tuned for the
demo compose setup. Several of them differ from the defaults compiled into the
binaries, which apply when you run a GUAC binary without a config file. Where
the two differ, the binary default is called out alongside the entry.

The keys that currently differ:

| Key            | `guac.yaml`             | Binary default                                           |
| -------------- | ----------------------- | -------------------------------------------------------- |
| `pubsub-addr`  | `nats://localhost:4222` | `nats://127.0.0.1:4222`                                  |
| `interval`     | `20m`                   | `5m`                                                     |
| `gql-debug`    | `true`                  | `false`                                                  |
| `db-address`   | not set                 | `postgres://guac:guac@0.0.0.0:5432/guac?sslmode=disable` |
| `arango-user`  | `root`                  | empty                                                    |
| `arango-pass`  | `test123`               | empty                                                    |
| `neo4j-user`   | `neo4j`                 | empty                                                    |
| `neo4j-pass`   | `s3cr3t`                | empty                                                    |
| `neptune-user` | `username`              | empty                                                    |

## Setting configuration values

Every key on this page can be set three ways. In order of precedence, highest
first:

1. **A command-line flag**, named after the key with a `--` prefix — for example
   `--db-address`.
2. **An environment variable**, named `GUAC_` followed by the key uppercased
   with dashes replaced by underscores — for example `GUAC_DB_ADDRESS`. This is
   usually the most convenient way to configure GUAC in containers.
3. **The `guac.yaml` file**, using the key as written on this page.

So the `db-address` key, the `--db-address` flag and the `GUAC_DB_ADDRESS`
environment variable all set the same value, and a flag passed on the command
line overrides both of the others:

```bash
# these three are equivalent
guacgql --db-address "postgres://guac:guac@localhost:5432/guac?sslmode=disable"

GUAC_DB_ADDRESS="postgres://guac:guac@localhost:5432/guac?sslmode=disable" guacgql

# or in guac.yaml:
#   db-address: postgres://guac:guac@localhost:5432/guac?sslmode=disable
```

## Database Configuration

The GraphQL server selects its graph backend with `gql-backend`. The binary
default is `keyvalue`, and the registered backend names are `keyvalue`,
`arango`, `ent`, `neo4j`, and `neptune`.

Backend-specific options are registered alongside the common `guacgql` flags, so
they can use the same command-line, environment-variable, or `guac.yaml`
configuration forms described above.

### Key-value backend

The `keyvalue` backend supports three stores: `memmap`, Redis, and TiKV.

- **kv-store**: `memmap`
  - **Description**: Selects the key-value store. Supported values are `memmap`,
    `redis`, and `tikv`.
  - **When to Change**: Use `redis` or `tikv` when the graph must persist
    outside the `guacgql` process.

{: .warning }

The default `memmap` store keeps graph data in an in-memory Go map. Data stored
there is lost when the `guacgql` process stops or restarts.

- **kv-redis**: `redis://user@localhost:6379/0`
  - **Description**: Experimental Redis connection string used when `kv-store`
    is `redis`.
  - **When to Change**: Point this at the Redis instance that should persist the
    key-value graph.

- **kv-tikv**: `127.0.0.1:2379`
  - **Description**: Experimental TiKV placement-driver address used when
    `kv-store` is `tikv`.
  - **When to Change**: Point this at the TiKV placement driver for your
    deployment.

For example, to use Redis:

```yaml
gql-backend: keyvalue
kv-store: redis
kv-redis: redis://user@redis:6379/0
```

### Ent config

- **db-driver**: `postgres`
  - **Description**: The driver used for the database connection.
  - **When to Change**: Modify if you need to switch to a different database
    driver.

- **db-address**: `postgres://guac:guac@postgres:5432/guac?sslmode=disable`
  - **Description**: The address for connecting to the PostgreSQL database. This
    key is not present in `guac.yaml`; the value shown is the one used by the
    compose setup. The binary default is
    `postgres://guac:guac@0.0.0.0:5432/guac?sslmode=disable`.
  - **When to Change**: Update if your database address changes or if you need
    to enable/disable SSL.

- **db-migrate**: `true`
  - **Description**: Indicates whether database migration is enabled.
  - **When to Change**: Set to `false` if you do not want automatic database
    migration.

### ArangoDB

Select this backend with `gql-backend: arango`.

- **arango-addr**: `http://localhost:8529`
  - **Description**: Address of the ArangoDB server.
  - **When to Change**: Update this when ArangoDB is hosted at another address.

- **arango-user**: `root` (binary default: empty)
  - **Description**: Username used to authenticate to ArangoDB.
  - **When to Change**: Set this to the user configured for your ArangoDB
    deployment.

- **arango-pass**: `test123` (binary default: empty)
  - **Description**: Password used to authenticate to ArangoDB.
  - **When to Change**: Replace the demo configuration value with the credential
    for your deployment.

### Neo4j

Select this backend with `gql-backend: neo4j`.

- **neo4j-addr**: `neo4j://localhost:7687`
  - **Description**: Address of the Neo4j server.
  - **When to Change**: Update this when Neo4j is hosted at another address.

- **neo4j-user**: `neo4j` (binary default: empty)
  - **Description**: Username used to authenticate to Neo4j.
  - **When to Change**: Set this to the user configured for your Neo4j
    deployment.

- **neo4j-pass**: `s3cr3t` (binary default: empty)
  - **Description**: Password used to authenticate to Neo4j.
  - **When to Change**: Replace the demo configuration value with the credential
    for your deployment.

- **neo4j-realm**: `neo4j`
  - **Description**: Authentication realm passed to the Neo4j driver.
  - **When to Change**: Change this only when your Neo4j authentication setup
    uses another realm.

### Neptune

Select this backend with `gql-backend: neptune`.

- **neptune-endpoint**: `localhost`
  - **Description**: Hostname of the Amazon Neptune database endpoint.
  - **When to Change**: Set this to the endpoint of your Neptune cluster.

- **neptune-port**: `8182`
  - **Description**: Port used for the Neptune connection.
  - **When to Change**: Change this if the cluster uses a different port.

- **neptune-region**: `us-east-1`
  - **Description**: AWS region used when signing requests to Neptune.
  - **When to Change**: Set this to the region that contains your Neptune
    cluster.

- **neptune-user**: `username` (binary default: empty)
  - **Description**: Username passed through to the Neo4j-compatible connection
    used by the Neptune backend.
  - **When to Change**: Set this if your deployment requires a specific user.

- **neptune-realm**: `neptune`
  - **Description**: Authentication realm used for the Neo4j-compatible
    connection to Neptune.
  - **When to Change**: Change this only when your authentication setup requires
    another realm.

## Pub/Sub Configuration

- **pubsub-addr**: `nats://localhost:4222` (binary default:
  `nats://127.0.0.1:4222`)
  - **Description**: The address of the NATS server for pub/sub messaging.
  - **When to Change**: Update if your NATS server is hosted elsewhere or uses a
    different port.

- **publish-to-queue**: `true`
  - **Description**: Whether to publish messages to the queue.
  - **When to Change**: Set to `false` if you do not want to use queue-based
    messaging.

## Blob Store Configuration

- **blob-addr**: `file:///tmp/blobstore?no_tmp_dir=true`
  - **Description**: The address of the blob store.
  - **When to Change**: Change to use a different storage backend, such as AWS
    S3 or Google Cloud Storage (examples).

## Certifier Configuration

- **interval**: `20m` (binary default: `5m`)
  - **Description**: The interval at which the certifier runs.
  - **When to Change**: Adjust based on how frequently you need certification
    checks.

- **last-scan**: `4`
  - **Description**: The number of hours since the last scan was run. A value of
    `0` means the scan will run on all packages/sources.
  - **When to Change**: Set to `0` for a full scan or adjust based on your
    scanning frequency needs.

- **certifier-batch-size**: `60000`
  - **Description**: The batch size for the package pagination query.
  - **When to Change**: Modify to optimize performance based on your system's
    capabilities.

- **certifier-latency**: `""`
  - **Description**: Artificial latency to throttle the certifier.
  - **When to Change**: Use to introduce delays if needed to manage load.

- **add-vuln-metadata**: `false`
  - **Description**: Whether the OSV certifier adds severity and other metadata
    to the vulnerabilities it ingests.
  - **When to Change**: Set to `true` if you want vulnerability metadata
    alongside the vulnerabilities themselves.

- #### Deps.dev Configuration
  - **deps-dev-latency**: `""`
  - **Description**: Artificial latency to throttle deps.dev.
  - **When to Change**: Adjust if you need to manage load on deps.dev queries.

## Ingestion Configuration

- **add-vuln-on-ingest**: `false`
  - **Description**: Whether to query vulnerabilities during ingestion.
  - **When to Change**: Set to `true` if you want to automatically check for
    vulnerabilities during data ingestion.

- **add-license-on-ingest**: `false`
  - **Description**: Whether to query licenses during ingestion.
  - **When to Change**: Set to `true` if you want to automatically check for
    licenses during data ingestion.

- **add-eol-on-ingest**: `false`
  - **Description**: Whether to query endoflife.date for end-of-life data during
    ingestion.
  - **When to Change**: Set to `true` if you want to automatically check for EOL
    data during data ingestion.

- **add-depsdev-on-ingest**: `false`
  - **Description**: Whether to query deps.dev for scorecards and source
    association data during ingestion.
  - **When to Change**: Set to `true` if you want to automatically collect
    deps.dev metadata during data ingestion.

## Collector-Subscriber Configuration

- **csub-addr**: `localhost:2782`
  - **Description**: The address for the Collector-Subscriber service.
  - **When to Change**: Update if your Collector-Subscriber service is hosted on
    a different address.

- **csub-listen-port**: `2782`
  - **Description**: The port on which the Collector-Subscriber service listens.
  - **When to Change**: Change if your Collector-Subscriber service uses a
    different port.

## GraphQL Configuration

- **gql-backend**: `keyvalue`
  - **Description**: The graph backend used by the GraphQL server. Registered
    values are `keyvalue`, `arango`, `ent`, `neo4j`, and `neptune`.
  - **When to Change**: Select the backend that matches your database
    deployment, then configure its backend-specific options in the Database
    Configuration section above.

- **gql-listen-port**: `8080`
  - **Description**: The port on which the GraphQL server listens.
  - **When to Change**: Change if your GraphQL server uses a different port.

- **gql-debug**: `true` (binary default: `false`)
  - **Description**: Whether to enable debug mode for the GraphQL server.
  - **When to Change**: Set to `false` in production environments for security.

- **gql-addr**: `http://localhost:8080/query`
  - **Description**: The address of the GraphQL server.
  - **When to Change**: Update if your GraphQL server is hosted elsewhere.

## REST API Configuration

- **rest-api-server-port**: `8081`
  - **Description**: The port on which the API server listens.
  - **When to Change**: Change if your API server uses a different port.

## Collector Configuration

- **service-poll**: `true`
  - **Description**: Whether the collector should poll services.
  - **When to Change**: Set to `false` if polling is not required.

- **use-csub**: `true`
  - **Description**: Whether to use the Collector-Subscriber service.
  - **When to Change**: Set to `false` if not using Collector-Subscriber.

## Logging Configuration

- **log-level**: `Info`
  - **Description**: The logging level for the application.
  - **When to Change**: Adjust to `Debug` for more detailed logs during
    development or troubleshooting.

## Metrics and Observability Configuration

For detailed documentation, see [Metrics and
Observability]({{ site.baseurl }}{% link guac/metrics.md %}).

- **enable-prometheus**: `false`
  - **Description**: Whether to enable the Prometheus metrics endpoint
    (`/metrics`).
  - **When to Change**: Set to `true` to enable scraping metrics by Prometheus
    or compatible collectors.

- **prometheus-port**: `9091`
  - **Description**: The port on which the Prometheus metrics server listens for
    standalone collectors and ingestors.
  - **When to Change**: Change if port `9091` conflicts with another service in
    your environment.

- **enable-otel**: `false`
  - **Description**: Whether to enable OpenTelemetry metrics and distributed
    tracing.
  - **When to Change**: Set to `true` to export traces and metrics to an
    OpenTelemetry collector.

- **OpenTelemetry Environment Variables**:
  - `OTEL_EXPORTER_OTLP_ENDPOINT`: Address of the OTel collector endpoint (e.g.
    `http://localhost:4317`).
  - `OTEL_EXPORTER_OTLP_INSECURE`: Set to `true` to allow insecure (non-TLS)
    gRPC connections.
  - `OTEL_SERVICE_NAME`: Service name identifier attached to exported traces and
    metrics.
