---
layout: page
title: End of Life certifier
permalink: /certifier-eol/
parent: "How GUAC components work together"
---

# EOL certifier

## Overview

The End of Life (EOL) certifier queries the
[endoflife.date](https://endoflife.date) service to identify packages in the
GUAC graph that are no longer supported. When a package has an entry in the EOL
data, this certifier creates a `HasMetadata` node with:

| Key         | Description                                         | Example                        |
| ----------- | --------------------------------------------------- | ------------------------------ |
| Product     | Product as listed in endolife.date                  | log4j                          |
| Cycle       | Release cycle                                       | 2                              |
| Version     | Version in GUAC                                     | TODO is this correct? Example? |
| IsEOL       | If the product has reached EOL                      | false                          |
| EOLDate     | Date the release reaches EOL                        | 2021-12-29                     |
| LTS         | If the release cycle is a long-term support version | false                          |
| Latest      | Latest release in the cycle                         | 2.24.3                         |
| ReleaseDate | Date of latest release in the cycle                 | 2024-12-10                     |

## Available Options

### Usage

Basic command syntax:

```bash
guacone  certifier  eol [options]
```

### Flags

| Flag                         | Description                                                                  | Default             |
| ---------------------------- | ---------------------------------------------------------------------------- | ------------------- |
| `--certifier-batch-size int` | Sets the batch size for pagination query for the certifier                   | 60000               |
| `--certifier-latency string` | Sets artificial latency on the certifier (e.g., m, h, s, etc.)               | Not enabled (empty) |
| `-h, --help`                 | Help for eol certifier                                                       |                     |
| `-l, --last-scan int`        | Hours since the last scan was run; if not set, runs on all packages/sources. | 4                   |

### Global Flags

| Flag                      | Description                                                                                        | Default                       |
| ------------------------- | -------------------------------------------------------------------------------------------------- | ----------------------------- |
| `--add-license-on-ingest` | If enabled, the ingestor will query and ingest clearly defined licenses (increases ingestion time) |                               |
| `--add-vuln-on-ingest`    | If enabled, the ingestor will query and ingest OSV for vulnerabilities (increases ingestion time)  |                               |
| `--csub-addr string`      | Address to connect to collect-sub service                                                          | "localhost:2782"              |
| `--csub-tls`              | Enable TLS connection to the server                                                                |                               |
| `--csub-tls-skip-verify`  | Skip verifying server certificate (for self-signed certificates)                                   |                               |
| `--gql-addr string`       | Endpoint used to connect to GraphQL server                                                         | "http://localhost:8080/query" |
| `--header-file string`    | A text file containing HTTP headers to send to the GQL server, in RFC 822 format                   |                               |
| `-i, --interval string`   | If polling, set interval (e.g., m, h, s, etc.)                                                     | "5m"                          |
| `-p, --poll`              | Sets the collector or certifier to polling mode                                                    |
