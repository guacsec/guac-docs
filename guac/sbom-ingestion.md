---
layout: page
title: Ingesting SBOMs into GUAC
parent: "How GUAC works"
permalink: /guac/ingesting-sboms/
redirect_from: /ingesting-sboms/
nav_order: 1
---

# Ingesting SBOMs into GUAC

## Overview

Software Bill of Materials (SBOM) ingestion is essential in GUAC to help track
and analyze dependencies, vulnerabilities, and software supply chain metadata.

## Supported document types

GUAC's ingestion pipeline handles SBOMs as well as other software supply chain
documents. The table below lists document types with registered ingestion
parsers. Some types are produced by GUAC collectors or certifiers and arrive
with their type already set rather than being detected from a user-supplied
file.

| Document type       | Description                                                               |
| ------------------- | ------------------------------------------------------------------------- |
| `SPDX`              | SPDX software bills of materials.                                         |
| `CycloneDX`         | CycloneDX software bills of materials.                                    |
| `DSSE`              | DSSE envelopes; embedded payloads are unpacked and processed recursively. |
| `SLSA`              | SLSA provenance statements carried in in-toto statements.                 |
| `ITE6VUL`           | GUAC vulnerability certification statements.                              |
| `ITE6EOL`           | GUAC end-of-life certification statements.                                |
| `ITE6REF`           | GUAC reference certification statements.                                  |
| `ITE6MALWARE`       | GUAC malware certification statements.                                    |
| `ITE6CD`            | ClearlyDefined legal and license certification statements.                |
| `SCORECARD`         | OpenSSF Scorecard documents.                                              |
| `DEPS_DEV`          | deps.dev documents used by GUAC.                                          |
| `CSAF`              | Common Security Advisory Framework documents.                             |
| `OPEN_VEX`          | OpenVEX documents.                                                        |
| `OPAQUE`            | Internal container type used while unpacking JSON Lines input.            |
| `INGEST_PREDICATES` | Direct GUAC graph predicates for tightly controlled environments.         |

{: .warning }

`INGEST_PREDICATES` bypasses the backing-attestation validation used by normal
ingestion. It is disabled by default and is registered by `guacone collect` only
when the `GUAC_DANGER` environment variable is set.

### JSON Lines

GUAC recognizes JSON Lines input. A JSON Lines document is treated as an
`OPAQUE` container, unpacked line by line, and each line is processed again as
an independent JSON document. Each line must therefore be valid JSON containing
a document type that GUAC can process.

### Compressed input

GUAC can decompress BZIP2 and Zstandard input before detecting the document
format and type. Files ending in `.bz2` and `.zst` are recognized by extension;
GUAC can also detect these encodings from their file signatures. After
decompression, the document continues through the normal format and type
detection pipeline.

## Ingestion Methods

### Manual Ingestion

Use the following command to ingest an SBOM file into GUAC for analysis and
tracking dependencies:

```bash
guacone collect files my-sbom.spdx.json
```

This method is ideal when working with specific files or testing new SBOMs
locally. File ingestion also works for directories using standard shell globs.

### Daemon-Mode Ingestion

When configured, GUAC operates in _daemon mode_, using collectors to poll for
new SBOMs at regular intervals.

To use daemon-mode ingestion effectively, ensure the following:

1. _Configure polling intervals_ to balance between frequency and system load.
2. _Verify connectivity_ between GUAC and the data source to avoid ingestion
   delays.
