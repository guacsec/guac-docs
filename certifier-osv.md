---
layout: page
permalink: /osv-certifier/
title: "OSV Certifier Documentation"
description: "Guide to using the OSV Certifier in GUAC for vulnerability management"

---

# OSV Certifier Documentation 

## Overview

The OSV Certifier component of [GUAC](https://guac.sh) (Graph for Understanding Artifact Composition) integrates with the [OSV (Open Source Vulnerability) database](https://osv.dev) to provide vulnerability insights for open-source dependencies. It enables security risk assessment through vulnerability identification in software dependencies.

## Key Features

- **Vulnerability Detection**: Scans dependencies using SBOMs and cross-references with OSV's vulnerability catalog
- **Automated Updates**: Regular synchronization with OSV for current vulnerability data
- **Comprehensive Reporting**: Structured reports showing vulnerabilities by dependency and severity

## Integration Details

### Data Collection Process

1. **Dependency Matching**: 
   - Parses SBOM files for package information
   - Extracts package names, versions, and ecosystem identifiers
   - Validates package metadata format

2. **API Querying**: 
   - Queries OSV using standardized package identifiers
   - Batch processing for multiple dependencies
   - Handles API rate limiting and retries

3. **Data Correlation**: 
   - Maps vulnerabilities to GUAC's dependency graph
   - Associates vulnerability data with package versions
   - Maintains relationship between dependencies

### Covered Ecosystems

The OSV Certifier enables vulnerability detection across several verified package ecosystems, including npm, PyPI, Maven, Go, Cargo, and NuGet. Additionally, it covers a wide range of ecosystems: AlmaLinux, Alpine, Android, Bitnami, crates.io, Curl, Debian GNU/Linux, Git (for C/C++), GitHub Actions, Haskell, Hex, the Linux kernel, OSS-Fuzz, Packagist, Pub, Python (CRAN and Bioconductor), Rocky Linux, RubyGems, SwiftURL, and Ubuntu OS.

### Feature Support

**Supported Features:**
- Public vulnerability data from OSV’s database
- Dependency version mapping against known vulnerabilities
- Analysis of version ranges
- Package identification through [PURL](https://github.com/package-url/purl-spec)
- Severity classification using [CVSS](https://www.first.org/cvss/)

**Unsupported Features:**
- Detection of private vulnerabilities
- Non-OSV-covered ecosystems
- Binary vulnerability scanning
- Custom vulnerability feeds

## Available Options

### Usage
Basic command syntax:
```bash
guacone certifier osv [options]
```

### Flags

| Flag                          | Description                                                                                         | Default               |
|-------------------------------|-----------------------------------------------------------------------------------------------------|-----------------------|
| `--certifier-batch-size int`  | Sets the batch size for pagination query for the certifier.                                       | 60000                 |
| `--certifier-latency string`  | Sets artificial latency on the certifier (e.g., m, h, s, etc.).                                  | Not enabled (empty)   |
| `-h, --help`                  | Help for osv                                                                                      |                       |
| `-l, --last-scan int`         | Hours since the last scan was run; if not set, runs on all packages/sources.                     | 4                     |

### Global Flags

| Flag                           | Description                                                                                         | Default                          |
|--------------------------------|-----------------------------------------------------------------------------------------------------|----------------------------------|
| `--add-license-on-ingest`      | If enabled, the ingestor will query and ingest clearly defined licenses.                          | Warning: Increases ingestion time |
| `--add-vuln-on-ingest`         | If enabled, the ingestor will query and ingest OSV for vulnerabilities.                           | Warning: Increases ingestion time |
| `--csub-addr string`           | Address to connect to collect-sub service.                                                         | "localhost:2782"                |
| `--csub-tls`                   | Enable TLS connection to the server.                                                               |                                  |
| `--csub-tls-skip-verify`      | Skip verifying server certificate (for self-signed certificates).                                  |                                  |
| `--gql-addr string`            | Endpoint used to connect to GraphQL server.                                                       | "http://localhost:8080/query"   |
| `--header-file string`         | A text file containing HTTP headers to send to the GQL server, in RFC 822 format.                 |                                  |
| `-i, --interval string`        | If polling, set interval (e.g., m, h, s, etc.).                                                  | "5m"                             |
| `-p, --poll`                   | Sets the collector or certifier to polling mode.                                                  |                                  |

## Output Format

### Vulnerability Report Fields

| Field | Description | Example |
|-------|-------------|---------|
| id | OSV vulnerability identifier | `OSV-2023-001` |
| package | Affected package name | `example-library` |
| version | Affected version | `1.2.3` |
| severity | Vulnerability severity | `High` |
| remediation | Fix instructions | `Update to version 1.2.4 or later` |

### Sample Output

```json
{
  "_type": "https://in-toto.io/Statement/v0.1",
  "subject": [
    {
      "uri": "pkg:npm/example-library@1.2.3"
    }
  ],
  "predicateType": "https://in-toto.io/attestation/vulns/v0.1",
  "predicate": {
    "scanner": {
      "uri": "osv.dev",
      "version": "0.0.14",
      "result": [
        {
          "id": "GHSA-rc38-5r82-hr3j"
        },
        {
          "id": "CVE-2023-12345"
        }
      ]
    },
    "metadata": {
      "scanStartedOn": "2023-06-06T06:15:28Z",
      "scanFinishedOn": "2023-06-06T06:15:28Z"
    }
  }
}
```

## Limitations

- Limited to vulnerabilities published in OSV
- May have incomplete data for certain ecosystems
- Does not detect issues in private/proprietary software
- Requires valid SBOM input
- Dependency on OSV API availability

## Additional Resources

- [GUAC Documentation](https://guac.sh)
- [OSV Database](https://osv.dev)
- [OSV API Documentation](https://osv.dev/docs)
- [PURL Specification](https://github.com/package-url/purl-spec)
- [CVSS Documentation](https://www.first.org/cvss/specification-document)

## Support

For issues and questions:
- [GUAC GitHub Issues](https://github.com/guacsec/guac/issues)
- [GUAC Slack Channel](https://openssf.slack.com/archives/C03U677QD46)
