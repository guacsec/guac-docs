---
layout: page
permalink: /guac/certifier-deps-dev/
redirect_from: /certifier-deps-dev/
title: "Deps.dev certifier"
parent: "How GUAC components work together"
description:
  "Guide to using the deps.dev certifier in GUAC for dependency and metadata
  enrichment"
---

# Deps.dev Certifier

## Overview

The deps.dev certifier component of [GUAC](https://guac.sh) (Graph for
Understanding Artifact Composition) integrates with
[Open Source Insights (deps.dev)](https://deps.dev) by Google to enrich the GUAC
supply chain knowledge graph. It automatically queries package metadata,
identifies source code repositories, attaches OpenSSF Scorecard security
evaluation metrics, and maps full dependency relationship trees for open-source
packages.

## How GUAC Pulls from deps.dev

GUAC connects directly to the deps.dev service via its high-performance gRPC API
(`api.deps.dev:443`).

When packages are ingested into GUAC or provided via the command line:

1. **Package URL (pURL) Translation**:
   - Parses package identifiers into standardized
     [Package URLs (pURLs)](https://github.com/package-url/purl-spec).
   - Translates the pURL (ecosystem type, namespace, name, version) into a
     deps.dev `VersionKey`.

2. **Metadata and Source Code Discovery**:
   - Queries `GetVersion` to fetch package version metadata and discover source
     repository URLs (`SOURCE_REPO` link).
   - If a source repository is identified, queries `GetProject` to retrieve
     [OpenSSF Scorecard](https://scorecard.dev) evaluation scores and security
     check results.

3. **Dependency Resolution**:
   - When configured with `--retrieve-dependencies`, calls `GetDependencies` to
     retrieve the package's complete dependency graph directly from deps.dev.

4. **GUAC Graph Node Creation**:
   - **`HasSourceAt`**: Associates the package (`Pkg`) with its source code
     repository (`Src`) with the justification `"collected via deps.dev"`.
   - **`CertifyScorecard`**: Ingests OpenSSF Scorecard ratings (aggregate score,
     checks, commit, version) for the source repository.
   - **`IsDependency`**: When dependency resolution is enabled, creates
     dependency edges between packages with the justification
     `"dependency data collected via deps.dev"`.
   - **`HasSBOM`**: Creates a top-level `HasSBOM` node for the package.

## Covered Ecosystems

The deps.dev certifier supports the primary package ecosystems indexed by
deps.dev:

- **npm** (`pkg:npm/...`)
- **PyPI** (`pkg:pypi/...`)
- **Go** (`pkg:golang/...`)
- **Maven** (`pkg:maven/...`)
- **Cargo** (`pkg:cargo/...`)
- **NuGet** (`pkg:nuget/...`)

{: .note } The pURL must contain an explicit package version (for example,
`pkg:npm/express@4.18.2`). Packages without a version cannot be resolved against
deps.dev and will be skipped.

## What is and is not Supported

### Supported Features

- **Source Repository Mapping**: Resolves package versions to upstream VCS
  repositories (`HasSourceAt`).
- **Scorecard Ingestion**: Automatically enriches source repositories with
  OpenSSF Scorecard security checks and scores (`CertifyScorecard`).
- **Transitive Dependency Resolution**: Automatically resolves and links full
  dependency trees (`IsDependency`) when `--retrieve-dependencies` is enabled.
- **Continuous Polling via CollectSub**: Works seamlessly with the CollectSub
  (`csub`) service to watch for newly ingested packages and query deps.dev
  automatically.
- **On-Demand Enrichment during Ingestion**: Can run automatically during
  document/SBOM ingestion with the `--add-depsdev-on-ingest` flag.
- **Manual CLI Ingestion**: Supports direct querying for specific pURLs via the
  CLI.
- **Rate Limiting and Throttling**: Built-in client rate limiting (150 RPS) with
  optional artificial latency (`--deps-dev-latency`).
- **Observability**: Exposes Prometheus metrics and OpenTelemetry traces.

### Limitations

- **Version Requirement**: Packages without an explicit version specified in the
  pURL are not queried.
- **Private/Proprietary Packages**: Only public open-source packages indexed by
  deps.dev are supported.
- **Unsupported Ecosystems**: Ecosystems not tracked by deps.dev cannot be
  resolved.
- **Network Access**: Requires outbound network access to `api.deps.dev:443`.
- **Ingestion Overhead**: Resolving deep transitive dependency trees
  (`--retrieve-dependencies`) or enabling `--add-depsdev-on-ingest` can increase
  ingestion time for large SBOMs.

## Invocation

GUAC integrates with deps.dev in three primary ways:

1. **Continuous Polling via Collect-Subscriber (Daemon Mode)**
   - The certifier/collector continuously monitors the CollectSub service for
     newly discovered pURLs from ingested SBOMs and queries deps.dev in the
     background.

2. **On-Demand during Ingestion**
   - Automatically queries deps.dev when ingesting documents or SBOMs by setting
     `--add-depsdev-on-ingest` (or `add-depsdev-on-ingest: true` in
     `guac.yaml`).

3. **Manual CLI Execution**
   - Directly query deps.dev for one or more specific package URLs using
     `guaccollect` or `guacone`.

### Usage

Basic command syntax:

```bash
# Using guaccollect
guaccollect deps_dev [flags] <purl1> <purl2>...

# Using guacone
guacone collect deps_dev [flags] <purl1> <purl2>...
```

### Core Flags

| Flag                      | Description                                                                        | Default             |
| ------------------------- | ---------------------------------------------------------------------------------- | ------------------- |
| `--retrieve-dependencies` | Query and collect full dependency relationships for the specified package(s).      | `false`             |
| `--deps-dev-latency`      | Sets artificial latency on deps.dev requests to manage load (e.g., `100ms`, `1s`). | Not enabled (empty) |
| `-p, --poll`              | Sets the collector/certifier to polling mode.                                      | `false`             |
| `--service-poll`          | Sets the collector/certifier to polling mode when using `guaccollect`.             | `true`              |
| `--use-csub`              | Use the CollectSub server to dynamically receive package datasources.              | `true`              |
| `--interval string`       | Polling interval when running in polling mode (e.g., `5m`, `10m`).                 | `5m`                |
| `--enable-prometheus`     | Enable Prometheus metrics HTTP handler.                                            | `false`             |
| `--prometheus-port int`   | Port to listen on for Prometheus metrics server.                                   | `9091`              |
| `--enable-otel`           | Enable OpenTelemetry metrics and distributed tracing.                              | `false`             |
| `-h, --help`              | Help for deps_dev command.                                                         |                     |

### Global Flags

| Flag                     | Description                                                                       | Default                                   |
| ------------------------ | --------------------------------------------------------------------------------- | ----------------------------------------- |
| `--csub-addr string`     | Address to connect to collect-sub service.                                        | `"localhost:2782"`                        |
| `--csub-tls`             | Enable TLS connection to the CollectSub server.                                   | `false`                                   |
| `--csub-tls-skip-verify` | Skip verifying CollectSub server certificate (for self-signed certificates).      | `false`                                   |
| `--gql-addr string`      | Endpoint used to connect to GraphQL server.                                       | `"http://localhost:8080/query"`           |
| `--pubsub-addr string`   | Address to connect to NATS pubsub service.                                        | `"nats://127.0.0.1:4222"`                 |
| `--blob-addr string`     | Address for blob storage.                                                         | `"file:///tmp/blobstore?no_tmp_dir=true"` |
| `--publish-to-queue`     | Enable publishing collected documents to the message queue.                       | `true`                                    |
| `--header-file string`   | A text file containing HTTP headers to send to the GQL server, in RFC 822 format. | Not set                                   |

## Usage Examples

### Query Specific Package URLs

Query deps.dev for metadata and source repository information for specific
packages:

```bash
# Query npm and Go packages
guaccollect deps_dev pkg:npm/express@4.18.2 pkg:golang/github.com/gin-gonic/gin@v1.9.0
```

```bash
# Using guacone CLI
guacone collect deps_dev pkg:npm/express@4.18.2
```

### Retrieve Full Dependency Graph

To also pull transitive dependencies and map dependency edges into the GUAC
graph:

```bash
guaccollect deps_dev --retrieve-dependencies pkg:npm/express@4.18.2
```

### Continuous Polling with CollectSub

Run the deps.dev certifier continuously to enrich packages as they are
discovered:

```bash
guaccollect deps_dev --use-csub --service-poll --interval=10m
```

### Ingestion-Time Enrichment

To automatically fetch deps.dev metadata during SBOM ingestion, enable the
`--add-depsdev-on-ingest` flag:

```bash
guacone ingest sbom /path/to/sbom.json --add-depsdev-on-ingest
```

Or configure in `guac.yaml`:

```yaml
add-depsdev-on-ingest: true
```

## Additional Resources

- [Open Source Insights (deps.dev)](https://deps.dev)
- [deps.dev API Documentation](https://docs.deps.dev/api/v3/)
- [OpenSSF Scorecard Project](https://github.com/ossf/scorecard)
- [Package URL (pURL) Specification](https://github.com/package-url/purl-spec)
- [GUAC GitHub Repository](https://github.com/guacsec/guac)
- [GUAC Slack Channel](https://openssf.slack.com/archives/C03U677QD46)
