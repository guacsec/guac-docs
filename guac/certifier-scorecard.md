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
   - Fetches pre-computed results from the OpenSSF Scorecard API
   - Falls back to computing the scorecard with the scorecard library when the
     API request fails
   - With `--compute`, computes the scores using the scorecard package

3. **Data Ingestion**:
   - Converts scorecard results to structured JSON format
   - Publishes results through GUAC's event stream for ingestion

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
| `--compute`                  | computes the scores using the scorecard package.               | false               |
| `-h, --help`                 | Help for scorecard                                             |                     |
| `--interval string`          | Polling interval (e.g., m, h, s, etc.)                         | 5m                  |
| `--service-poll`             | Enable polling mode                                            | false               |

### Global Flags

| Flag                   | Description                                                                      | Default                                 |
| ---------------------- | -------------------------------------------------------------------------------- | --------------------------------------- |
| `--gql-addr string`    | Endpoint used to connect to GraphQL server                                       | `http://localhost:8080/query`           |
| `--pubsub-addr string` | Address to connect to NATS pubsub service                                        | `nats://localhost:4222`                 |
| `--blob-addr string`   | Address for blob storage                                                         | `file:///tmp/blobstore?no_tmp_dir=true` |
| `--header-file string` | A text file containing HTTP headers to send to the GQL server, in RFC 822 format | Not set                                 |
| `--publish-to-queue`   | Enable publishing to the message queue                                           | true                                    |

## Usage Examples

### Basic Usage

```bash
# Set GitHub token (used by the computation fallback)
export GITHUB_AUTH_TOKEN=your_github_token
```

```bash
# Run the scorecard certifier
guaccollect scorecard
```

### Computation by skipping API

```bash
# computes the scores using the scorecard package
# Requires GITHUB_AUTH_TOKEN; the certifier exits at startup if it is unset.
guaccollect scorecard --compute
```

Useful for air-gapped networks or repositories the public API has never scanned.

### Polling Mode

```bash
# Enable polling with a custom interval
guaccollect scorecard \
  --service-poll \
  --interval=10m
```

## Prerequisites

### GitHub Token Setup

1. Create a GitHub Personal Access Token

2. Set environment variable:
   ```bash
   export GITHUB_AUTH_TOKEN=ghp_your_token_here
   ```

## Limitations

- Currently supports GitHub repositories only
- Requires valid commit SHA or tag reference
- Results depend on repository accessibility and structure
- The computation fallback requires a GitHub authentication token, is slower for
  large repositories, and may hit GitHub API rate limits at high volume

## Error Handling

Common error scenarios and solutions:

### Authentication Errors

```
Error: GITHUB_AUTH_TOKEN is not set
```

**Solution**: Set the `GITHUB_AUTH_TOKEN` environment variable with a valid
GitHub token. Without `--compute` this is only a warning at startup, since the
API path needs no token; with `--compute` it is a hard error.

### Rate Limiting

```
Error: API returned status 429: Rate limit exceeded
```

**Solution**: Try reducing batch size using `--certifier-batch-size`.

## Additional Resources

- [OpenSSF Scorecard Project](https://github.com/ossf/scorecard)
- [Scorecard Checks Documentation](https://github.com/ossf/scorecard/blob/main/docs/checks.md)
