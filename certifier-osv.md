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
guac osv-certify --sbom <path> [options]
```

Available options:
```bash
Options:
  --sbom PATH        Path to SBOM file (required)
  --format FORMAT    SBOM format: cyclonedx|spdx (default: auto-detect)
  --output PATH     Output file path (default: stdout)
  --severity LEVEL  Minimum severity: critical|high|medium|low (default: low)
```

Example usage:
```bash
# Basic scan
guac osv-certify --sbom ./sboms/my_project_sbom.json

# Specify format and severity
guac osv-certify --sbom ./sbom.xml --format cyclonedx --severity high
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
