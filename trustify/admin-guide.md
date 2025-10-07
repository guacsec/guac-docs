---
title: Administration Guide
layout: page
permalink: /trustify/concepts/
nav_order: 2
parent: Trustify Docs
has_children: true
---

# Administration Guide

This administration guide gives you a better understanding on Trustify’s
deployment options, installing the Trustify services locally, and reference
information for Trustify’s OpenAPI structures.

Trustify relies on external services for storage and authentication. Trustify’s
services can use Amazon’s S3 APIs to store object data. You do not need to run
Trustify on Amazon Web Services (AWS), because other cloud vendors, such as
Google Cloud Storage (GCS) and MinIO, implement Amazon’s S3 API.

It also is possible to run Trustify on bare metal. Either on a server, or a
local desktop/laptop, for trying it out.

## Deployment options

You have several deployment options to choose from for running Trustify.

### Bare metal

Running Trustify on bare-metal servers requires you to download the `trustd`
binary from source found on the
[Trustify releases](https://github.com/guacsec/trustify/releases) page.

There currently exist two variants of this binary, `trustd` and `trustd-pm`. The
`-pm` version includes a few embedded services, such as PostgreSQL and an OIDC
issuer, which make it easier to run Trustify locally for trying it out. However,
this might be insecure, dangerous, and is not production ready. Still, it is
great for trying it out and demoing it without too much trouble.

### Container image

Trustify services are available in an image, `ghcr.io/guacsec/trustd`.

### Kubernetes

Since Kubernetes runs containers, running Trustify on Kubernetes is easy! We do
not provide any out-of-the-box charts or an installation script, since many of
the configuration options are specific to the running environment.

We keep a Helm chart in the https://github.com/guacsec/trustify-helm-charts to
deploy a Trustify instance. You can use this Helm chart as a starting point for
your specific environment.

The [Kubernetes section]({%link trustify/kubernetes.md %}) has more details.

### Data Migration

If you are already using the Trustification project, you might want to migrate
your SBOMs to Trustify (vulnerability and advisory information are automatically
ingested into the Trustify database).

As Trustification project uses S3 compatible storage to store original SBOMs
files, the best way to migrate data is to ingest them from the appropriate S3
bucket.

At the moment there is no automated tool to achieve this, but the process is
simple. First, download SBOMs from the S3 bucket to the local folder:
`aws s3 sync s3://bombastic-prod/data/ /tmp/sboms/`

Now you can use any tool at your disposal to traverse this directory and ingest
the SBOMs into Trustify, by posting them to the SBOM upload API endpoint. For
example, [CSAF Walker library](https://github.com/ctron/csaf-walker) contains
the `sbom` command line tool that can be used for this job:
`sbom scoop http://localhost:8080/api/v2/sbom /tmp/sboms/`

In the future we will provide more options to ingest SBOMs directly from
different cloud object storage providers.
