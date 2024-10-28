layout: doc
permalink: /osv-certifier/
title: "OSV Certifier Documentation"
description: "Guide to using the OSV Certifier in GUAC for vulnerability management"
categories: [certification, vulnerability-management, GUAC]
tags: [OSV, GUAC, software-supply-chain, security, SBOM]
---

# OSV Certifier Documentation 

## Overview

The OSV Certifier component of [GUAC](https://guac.sh) (Graph for Understanding Artifact Composition) integrates with the [OSV (Open Source Vulnerability) database](https://osv.dev) to provide vulnerability insights for open-source dependencies. It enables security risk assessment through vulnerability identification in software dependencies.

## Architecture

```
┌──────────────┐     ┌───────────────┐     ┌────────────┐
│  SBOM Input  │────>│  OSV Certifier│────>│  OSV API   │
└──────────────┘     └───────────────┘     └────────────┘
                            │
                            ▼
                    ┌───────────────┐
                    │  GUAC Graph   │
                    └───────────────┘
```

## Key Features

- **Vulnerability Detection**: Scans dependencies using SBOMs and cross-references with OSV's vulnerability catalog
- **Automated Updates**: Regular synchronization with OSV for current vulnerability data
- **Comprehensive Reporting**: Structured reports showing vulnerabilities by dependency and severity
- **SBOM Format Support**: 
  - [CycloneDX](https://cyclonedx.org) (JSON, XML)
  - [SPDX](https://spdx.dev) (JSON, YAML)

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

### Supported Package Ecosystems

The OSV Certifier supports vulnerability detection across these verified ecosystems:

| Ecosystem | Identifier Format | Example |
|-----------|------------------|---------|
| npm | `pkg:npm/{name}@{version}` | `pkg:npm/lodash@4.17.21` |
| PyPI | `pkg:pypi/{name}@{version}` | `pkg:pypi/requests@2.28.1` |
| Maven | `pkg:maven/{group}/{artifact}@{version}` | `pkg:maven/org.apache.logging.log4j/log4j-core@2.14.1` |
| Go | `pkg:golang/{path}@{version}` | `pkg:golang/golang.org/x/text@v0.3.7` |
| Cargo | `pkg:cargo/{name}@{version}` | `pkg:cargo/serde@1.0.152` |
| NuGet | `pkg:nuget/{name}@{version}` | `pkg:nuget/Newtonsoft.Json@13.0.1` |

### Feature Support

#### Supported
- Public vulnerabilities from OSV's database
- Dependency version mapping with known vulnerabilities
- Version range analysis
- [PURL](https://github.com/package-url/purl-spec) package identification
- Severity classification using [CVSS](https://www.first.org/cvss/) scores

#### Unsupported
- Private vulnerability detection
- Ecosystems not covered by OSV
- Binary vulnerability scanning
- Custom vulnerability feeds

## Available Options

### Usage
Basic command syntax:
```bash
guacone certifier osv --sbom <path> [options]
```

### Flags
```bash
Flags:
      --certifier-batch-size int   sets the batch size for pagination query for the certifier (default 60000)
      --certifier-latency string   sets artificial latency on the certifier. Defaults to empty string (not enabled) but can set m, h, s...etc
  -h, --help                       help for osv
  -l, --last-scan int              hours since the last scan was run. If not set, run on all packages/sources (default 4)
```
### Global Flags:
```bash
      --add-license-on-ingest   if enabled, the ingestor will query and ingest clearly defined for licenses. Warning: This will increase ingestion times
      --add-vuln-on-ingest      if enabled, the ingestor will query and ingest OSV for vulnerabilities. Warning: This will increase ingestion times
      --csub-addr string        address to connect to collect-sub service (default "localhost:2782")
      --csub-tls                enable tls connection to the server
      --csub-tls-skip-verify    skip verifying server certificate (for self-signed certificates for example)
      --gql-addr string         endpoint used to connect to graphQL server (default "http://localhost:8080/query")
      --header-file string      a text file containing HTTP headers to send to the GQL server, in RFC 822 format
  -i, --interval string         if polling set interval, m, h, s, etc. (default "5m")
  -p, --poll                    sets the collector or certifier to polling mode
```

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

### Remediation Process

1. **Prioritize**: 
   - Review vulnerability severity
   - Assess impact on application
   - Consider dependency update complexity

2. **Update**: 
   - Follow remediation guidance
   - Test compatibility of new versions
   - Update dependency declarations

3. **Verify**: 
   - Rescan with OSV Certifier
   - Confirm vulnerability resolution
   - Update SBOM documentation

## Error Handling

Common error scenarios and resolutions:

| Error | Cause | Resolution |
|-------|-------|------------|
| Invalid SBOM | Malformed SBOM file | Validate SBOM against schema |
| API Timeout | OSV API unresponsive | Retry with backoff |
| Unknown Format | Unsupported SBOM format | Use supported format |
| Invalid Package | Malformed package URL | Check PURL syntax |

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
- [CycloneDX Specification](https://cyclonedx.org/specification/overview/)
- [SPDX Specification](https://spdx.dev/specifications/)
- [PURL Specification](https://github.com/package-url/purl-spec)
- [CVSS Documentation](https://www.first.org/cvss/specification-document)

## Support

For issues and questions:
- [GUAC GitHub Issues](https://github.com/guacsec/guac/issues)
- [GUAC Slack Channel](https://openssf.slack.com/archives/C03U677QD46)
