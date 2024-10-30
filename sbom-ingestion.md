---
layout: page
title: Ingesting SBOMs into GUAC
permalink: /ingesting-sboms/
---

### Overview

Software Bill of Materials (SBOM) ingestion is essential in GUAC (Graph for
Understanding Artifact Composition) to help track and analyze dependencies,
vulnerabilities, and software supply chain metadata. This documentation explains
how to ingest SBOMs in GUAC, covering:

- **Supported formats and limitations**
- **Ingestion methods**
- **Daemon-mode ingestion**
- **Manual vs. automatic ingestion**

---

### Supported SBOM Formats

GUAC supports a variety of SBOM formats, making it compatible with several tools
and standards:

- **SPDX**: A widely used open standard for software package metadata.
- **CycloneDX**: An SBOM specification built for security use cases.

---

### Ingestion Methods

1. **Manual Ingestion**:

   - Use the following command to ingest an SBOM file into GUAC for analysis and
     tracking dependencies:
   - Example:
     ```bash
     guacone collect files my-sbom.spdx.json
     ```
   - This method is ideal when working with specific files or testing new SBOMs
     locally.

2. **Daemon-Mode Ingestion (Polling Collectors)**:
   - When configured, GUAC operates in **daemon mode**, using collectors (like
     GCS) to poll for new SBOMs at regular intervals.

---

### Recommended Configuration for Daemon Mode

To use daemon-mode ingestion effectively, ensure the following:

1. **Configure polling intervals** to balance between frequency and system load.
2. **Verify connectivity** between GUAC and the data source to avoid ingestion
   delays.

```

```
