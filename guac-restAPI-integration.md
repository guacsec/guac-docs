# GUAC REST API Documentation

## Overview

The GUAC REST API provides an alternative to the GUAC GraphQL API, offering simplified access to advanced queries. It is ideal for users or developers who prefer REST endpoints over GraphQL queries and need to integrate GUAC services into their own tools and scripts.

## Base URL

```
http://<host>:<port>/
```

## Endpoints

### 1. /healthz

Description: Health check to ensure the server is running correctly.

Method: `GET`

Response:

```json
"Healthy"
```

### 2. /analysis/dependencies

Description: Identify the most important dependencies by frequency or score.

Method: `GET`

Query Parameters:

- PaginationSpec (optional): Pagination details like page size and cursor.
- sort (required): Sort order of the packages. Options:
  - `'frequency'`: Packages with the most dependents.
  - `'scorecard'`: Packages with the lowest OpenSSF scorecard score.

Responses:

- 200 OK:
  ```json
  [
    { "Name": "example/dependency1", "DependentCount": 100 },
    { "Name": "example/dependency2", "DependentCount": 50 }
  ]
  ```

### 3. /v0/package/{purl}

Description: Retrieve all package URLs (purls) related to the provided `purl`.

Method: `GET`  
Path Parameter:

- purl (required): A URL-encoded Package URL (purl).

Response:

```json
{
  "PaginationInfo": { "NextCursor": "abc123", "TotalCount": 2 },
  "PurlList": ["pkg:foo/bar@1.0", "pkg:foo/bar@2.0"]
}
```

### 4. /v0/package/{purl}/vulns

Description: Get vulnerabilities associated with the given `purl` and its dependencies.

Method: `GET`  
Path Parameter:

- purl (required): A URL-encoded Package URL.

Query Parameter:

- includeDependencies (optional, default: `false`): Include vulnerabilities of dependencies.

Response:

```json
[
  {
    "package": "pkg:foo/bar",
    "vulnerability": { "type": "CVE", "vulnerabilityIDs": ["CVE-2024-0001"] }
  }
]
```

### 5. /v0/artifact/{digest}/dependencies

Description: Retrieve dependencies associated with a specific digest.

Method: `GET`  
Path Parameter:

- digest (required): Digest of the artifact in `<algorithm:digest>` format.

Response:

```json
{
  "PurlList": ["pkg:foo/bar@1.0", "pkg:foo/bar@2.0"]
}
```

### 6. GET /v0/package/{purl}/dependencies
Retrieve the dependencies associated with a specific Package URL (purl). If a partial purl is provided, all associated purls and dependencies are included.

Request Example:
```bash
GET /api/v1/package/{purl}/dependencies
```

Response:
```json
{
  "dependencies": ["pkg:foo/bar@1.0", "pkg:foo/bar@2.0"]
}
```

### 7. GET /v0/artifact/{digest}/vulns
Retrieve vulnerabilities related to a specific artifact digest.

Request Example:
```bash
GET /api/v1/artifact/{digest}/vulns
```

Response:
```json
{
  "vulnerabilities": [
    {
      "id": "CVE-2023-12345",
      "description": "Sample vulnerability description.",
      "severity": "High"
    }
  ]
}
```


#### Vulnerability Schema

```json
{
  "package": "pkg:foo/bar",
  "vulnerability": {
    "type": "CVE",
    "vulnerabilityIDs": ["CVE-2024-0001"]
  },
  "metadata": {
    "scannerUri": "example-scanner",
    "timeScanned": "2024-10-26T10:00:00Z"
  }
}
```

## Errors handling

- 400 Bad Request: Invalid input or parameters.
- 500 Internal Server Error: Unexpected server error.
- 502 Bad Gateway: Backend connection issue.
- 404 Page Not Found

## Design Considerations

- Design of the REST API: https://github.com/guacsec/guac/issues/1544
- Infrastructure: Runs as a new binary under `cmd/rest` in the GUAC project, built using the existing Makefile setup

## Development & Contribution

The REST API is open for improvements. Developers can contribute by:

- Reporting issues via https://github.com/guacsec/guac/issues.
- Reviewing or expanding the API design through pull requests.

Check https://github.com/guacsec/guac/tree/main/cmd/guacrest for further details on Information on the Experimental guacrest API
