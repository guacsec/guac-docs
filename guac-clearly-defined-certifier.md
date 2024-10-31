---
layout: page
title: ClearlyDefined certifier
permalink: /certifier-clearlydefined/
---

### Overview

GUAC (Graph for Understanding Artifact Composition) integrates with
**ClearlyDefined** (https://clearlydefined.io/?sort=releaseDate&sortDesc=true)
to enhance supply chain transparency by retrieving accurate license data for
software dependencies. This functionality helps organizations make informed
decisions about software licenses when managing their dependencies.

### How GUAC Pulls from ClearlyDefined

GUAC accesses ClearlyDefined data through a **certifier process** instead of a
collector model. This choice ensures that queries are **re-run on a scheduled
basis** to reflect any updated license information, given that new software
releases can alter licensing data over time. The certifier’s role complements
GUAC’s use of SBOM (Software Bill of Materials) data, allowing it to validate or
augment license details.

- **Invocation**:

  - The certifier runs periodically or can be triggered during **SBOM
    ingestion** to pull data directly from ClearlyDefined.
  - However, including ClearlyDefined queries during SBOM ingestion may **slow
    down processing**, especially for larger datasets.
  - For further details visit
    https://docs.guac.sh/guac-configuration/#certifier-configuration.

- **Mapping and Query System**:
  - ClearlyDefined identifies software using **coordinates**, which GUAC maps to
    **pURLs (package URLs)**. A custom mapping library facilitates this
    conversion to align the two systems.

### What is and is not Supported

1. **Supported Features**:

   - GUAC retrieves detailed license data from ClearlyDefined.
   - Both SBOM-derived and ClearlyDefined license data are preserved, allowing
     users to make manual decisions in case of conflicts.
   - **CertifyLegal nodes** are created for both sources, ensuring complete
     graph observability.

2. **Limitations**:
   - **Batched queries** are not supported by ClearlyDefined’s API, which can
     result in rate limits and slower performance for large dependency graphs.
   - The system does not automatically resolve conflicting data between SBOMs
     and ClearlyDefined; users must determine the most trustworthy source
     themselves.

### Invocation Process

GUAC integrates with ClearlyDefined in three ways:

1. **Scheduled Certifier Execution**

   - Automatically runs at specified intervals to keep the dependency data up to
     date.

2. **On-Demand during SBOM Ingestion**

   - Queries ClearlyDefined in real time during the ingestion of SBOMs, which
     may slow down the processing.

3. **Manual Command Line Execution**
   - This method provides greater control, allowing users to run queries
     whenever needed.

---

Below is a table of the supported **command-line arguments** for the
**ClearlyDefined certifier**. These flags allow users to control input, output,
and execution behavior.

| **Argument**           | **Description**                                 | **Example**                 |
| ---------------------- | ----------------------------------------------- | --------------------------- |
| `--help`               | Displays help message.                          | `--help`                    |
| `--input <path>`       | Specifies input directory or file.              | `--input ./sboms`           |
| `--output <path>`      | Sets output directory.                          | `--output ./results`        |
| `--format <type>`      | Sets output format (e.g., `json`, `spdx`).      | `--format json`             |
| `--log-level <lvl>`    | Adjusts log level (`info`, `debug`, `error`).   | `--log-level debug`         |
| `--poll-interval <n>`  | Sets polling interval in minutes (daemon mode). | `--poll-interval 10`        |
| `--certify-only`       | Runs only certification, skipping ingestion.    | `--certify-only`            |
| `--collector <type>`   | Specifies collector (e.g., `gcs`, `github`).    | `--collector gcs`           |
| `--daemon`             | Runs in daemon mode for continuous polling.     | `--daemon`                  |
| `--config <path>`      | Loads custom configuration file.                | `--config ./config.yaml`    |
| `--api-endpoint <url>` | Sets API endpoint for results.                  | `--api-endpoint http://...` |

---

### Key Notes:

- **Daemon mode**: When invoked with `--daemon`, the certifier continuously
  polls the specified collector (like `gcs`) to fetch updates.
- **Poll interval**: Default polling may be every 5 or 10 minutes, configurable
  via the `--poll-interval` argument.
- **Formats**: Supported output formats include `json`, `spdx`, or other
  standards recognized by the tool.
