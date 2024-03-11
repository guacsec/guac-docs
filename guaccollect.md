---
layout: page
title: Ingesting data with GUACCollect
permalink: /guaccollect/
parent: GUAC demos
grand_parent: Getting started with GUAC
nav_order: 2
---

# Ingesting data with GUACCollect

GUACCollect is a command-line tool within the GUAC ecosystem designed for collecting and ingesting software bill of materials (SBOMs), attestations, and other metadata documents from various sources. This tool supports a wide range of data sources, including GitHub, S3, Google Cloud Storage (GCS), and OCI images, making it a versatile choice for enhancing the visibility and security of your software supply chain.

## Requirements

- A fresh copy of the [GUAC service infrastructure through Docker Compose]({{
  site.baseurl }}{%link setup.md %}). Including the `guacone` binary in your path
  and [GUAC Data](https://github.com/guacsec/guac-data/archive/refs/heads/main.zip)
  extracted to `guac-data-main`.
- Access to data sources: Depending on what sources you're collecting from, S3, GitHub, GCS, or OCI images.

## Use Cases

GUACCollect enables users to:

- **Collect Metadata from GitHub**: Fetch metadata documents from GitHub releases or workflows.
- **Ingest from Cloud Storage**: Support for S3, GCS, and other cloud storage solutions to ingest SBOMs and attestations.
- **Download from OCI Images**: Retrieve SBOMs and attestations embedded in OCI images.
- **File System Collection**: Collect documents directly from a specified file path on your system.

## Usage

GUACCollect offers a modular design with separate commands for each data source. Here are some examples:

### GitHub

```bash
./guaccollect github --github-mode release <release_url1> <release_url2>
./guaccollect github --github-mode workflow <owner>/<repo>
```

### S3 Compatible Storage

```bash
./guaccollect s3 --s3-url <s3_url> --s3-bucket <bucket_name> --poll
```

### Google Cloud Storage (GCS)

```bash
./guaccollect gcs <bucket_name> --gcs-credentials-path /path/to/credentials.json
```

### OCI Images

```bash
./guaccollect image <image_path1> <image_path2>
```

### Files

```bash
./guaccollect files <file_path>
```

![guaccollect graph](assets/images/guaccollectGraph.png)

## Configuration

GUACCollect supports various configuration flags for advanced usage. Use the `--help` flag to explore all options:

```bash
./guaccollect <command> --help
```

## Demo

To collect metadata from GitHub using GUACCollect, you'll need to specify the mode of collection release or workflow and provide the necessary GitHub URLs or repository details. Here's a step-by-step example for the workflow mode:

### Collecting from GitHub Workflows

To collect metadata from GitHub workflows, use the `github` command with the `--github-mode workflow` flag. Specify the owner and repository name in the format `<owner>/<repo>`. For example:

```bash
./guaccollect github --github-mode=workflow guacsec/guac-test
```

This command collects metadata from the workflows of the specified GitHub repository.

## Conclusion

GUACCollect is a key tool in the GUAC ecosystem for collecting and ingesting metadata documents, enhancing the visibility and security of your software supply chain. With its support for multiple data sources and flexible configuration, GUACCollect simplifies the process of building a comprehensive view of your software artifacts and their associated metadata.

For more detailed information, refer to the official [GUAC documentation](https://docs.guac.sh/).