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

## Supported SBOM Formats

GUAC supports a variety of SBOM formats, making it compatible with several tools
and standards:

- **SPDX**: A widely used open standard for software package metadata.
- **CycloneDX**: An SBOM specification built for security use cases.

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
