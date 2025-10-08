---
layout: page
permalink: /guac/scorecard-certifier/
title: "Scorecard certifier"
parent: "How GUAC components work together"
description:
  "Guide to using the Scorecard Certifier in GUAC for security assessment of
  open source repositories"
---

# Scorecard Certifier

## Overview

The Scorecard Certifier component of [GUAC](https://guac.sh) (Graph for
Understanding Artifact Composition) integrates with the
[OpenSSF Scorecard](https://github.com/ossf/scorecard) project to provide
comprehensive security risk assessments for open-source repositories. It
evaluates repositories against industry security best practices and provides
actionable security insights.

## Key Features

- **Security Risk Assessment**: Evaluates repositories using 19 comprehensive
  security checks

- **Automated Scoring**: Provides numerical scores (0-10) for each security
  check

- **Comprehensive Analysis**: Covers code review practices, dependency
  management, vulnerability handling, and more

- **Integration with GUAC**: Seamlessly connects scorecard data to GUAC's
  software supply chain graph

## Data Collection Process

1. **Repository Identification**:
   - Source repositories of packages are identified by GUAC
   - Supports GitHub repositories with commit SHA or tag references
   - Validates repository metadata and accessibility

2. **Scorecard Evaluation**:
   - Runs security checks against target repositories
   - Query mode: Fetches pre-computed results from OpenSSF Scorecard API
   - Compute mode: Executes scorecard checks using the scorecard library and
     computes the result

3. **Data Ingestion**:
   - Converts scorecard results to structured JSON format
   - Publishes results through GUAC's event stream for ingestion

## Operation Modes

The scorecard certifier supports two modes: **Query mode** (default) fetches
pre-computed results from the OpenSSF Scorecard API. It's fast, requires no authentication,
but only covers repositories already analyzed by OpenSSF. **Compute mode** runs
scorecard checks using the scorecard library, and computes the score. It provides complete
coverage of any GitHub repository, but requires a GitHub token and uses more resources.

## Available Options

### Usage

Basic command syntax:

```bash
guaccollect scorecard [options]
```

### Core Flags

| Flag                         | Description                                                    | Default             |
| ---------------------------- | -------------------------------------------------------------- | ------------------- |
| `--certifier-batch-size int` | Sets the batch size for pagination query for the certifier     | 60000               |
| `--certifier-latency string` | Sets artificial latency on the certifier (e.g., m, h, s, etc.) | Not enabled (empty) |
| `-h, --help`                 | Help for scorecard                                             |                     |
| `--interval string`          | Polling interval (e.g., m, h, s, etc.)                         | 5m                  |
| `--service-poll`             | Enable polling mode                                            | false               |

### Scorecard-Specific Flags

| Flag                        | Description                                                     | Default |
| --------------------------- | --------------------------------------------------------------- | ------- |
| `--scorecard-mode`          | Scorecard modes: 'query' (default) or 'compute'               | `query` |
| `--scorecard-http-timeout`  | HTTP timeout for API requests when using 'query' mode         | `30s`   |

### Global Flags

| Flag                   | Description                                                                      | Default                                 |
| ---------------------- | -------------------------------------------------------------------------------- | --------------------------------------- |
| `--gql-addr string`    | Endpoint used to connect to GraphQL server                                       | `http://localhost:8080/query`           |
| `--pubsub-addr string` | Address to connect to NATS pubsub service                                        | `nats://localhost:4222`                 |
| `--blob-addr string`   | Address for blob storage                                                         | `file:///tmp/blobstore?no_tmp_dir=true` |
| `--header-file string` | A text file containing HTTP headers to send to the GQL server, in RFC 822 format | Not set                                 |
| `--publish-to-queue`   | Enable publishing to the message queue                                           | true                                    |

## Usage Examples

### Query Mode (Default)

```bash
# Use query mode with default settings
guaccollect scorecard
```

```bash
# Use query mode with custom HTTP timeout
guaccollect scorecard \
  --scorecard-mode=query \
  --scorecard-http-timeout=60s
```

### Compute Mode

```bash
# Set GitHub token (required for compute mode)
export GITHUB_AUTH_TOKEN=your_github_token
```

```bash
# Use compute mode (scorecard library)
guaccollect scorecard \
  --scorecard-mode=compute
```

### Polling Mode

```bash
# Enable polling with custom interval
guaccollect scorecard \
  --service-poll \
  --interval=10m \
  --scorecard-mode=query
```

## Prerequisites

### GitHub Token Setup (Compute Mode Only)

1. Create a GitHub Personal Access Token

2. Set environment variable:
   ```bash
   export GITHUB_AUTH_TOKEN=ghp_your_token_here
   ```

## Limitations

### Query Mode Limitations

- Limited to repositories already analyzed by OpenSSF Scorecard
- No control over check configuration

### Compute Mode Limitations

- Requires GitHub authentication token
- Slower execution for large repositories
- May hit GitHub API rate limits with high volume

### General Limitations

- Currently supports GitHub repositories only
- Requires valid commit SHA or tag reference
- Results depend on repository accessibility and structure

## Error Handling

Common error scenarios and solutions:

### Authentication Errors (Compute Mode)

```
Error: GITHUB_AUTH_TOKEN is not set
```

**Solution**: Set the `GITHUB_AUTH_TOKEN` environment variable with a valid
GitHub token.

### Repository Not Found (Query Mode)

```
Error: repository not found in scorecard database
```

**Solution**: Repository may not be analyzed by OpenSSF Scorecard yet. Consider
using compute mode or wait for API coverage.

### Rate Limiting

```
Error: API returned status 429: Rate limit exceeded
```

**Solution**: Try reducing batch size using `--certifier-batch-size`.

## Additional Resources

- [OpenSSF Scorecard Project](https://github.com/ossf/scorecard)
- [Scorecard Checks Documentation](https://github.com/ossf/scorecard/blob/main/docs/checks.md)