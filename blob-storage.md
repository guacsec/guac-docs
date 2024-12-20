---
layout: page
title: Using blob storage with GUAC
parent: "Ingesting SBOMs into GUAC"
permalink: /ingesting-blob/
nav_order: 1
---

# Using blob storage with GUAC

GUAC can ingest files from blob storage using
[gocloud/blob](http://gocloud.dev/blob). The collector can download one item
from the storage, all items from a folder, a whole bucket or listen to storage
events using sqs/kafka (poll) and download the files as they are uploaded.

{: .note } The [GUAC Helm Charts](https://github.com/kusaridev/helm-charts)
maintained by Kusari includes [MinIO](https://charts.min.io/), an S3-compatible
blob store server.

## Amazon S3 and compatible

`guaccollect` supports blob storage compatible with the Amazon S3 API. This
section includes a non-exhaustive set of example usage.

To ingest from an AWS bucket named "guac-test":

```bash
guaccollect s3 --s3-bucket guac-test --s3-region eu-north-1
```

To ingest a folder named "sboms" contained in an AWS bucket named "guac-test":

```bash
guaccollect s3 --s3-bucket guac-test --s3-region eu-north-1 --s3-path sboms/
```

To ingest from an S3-compatible min.io bucket named "guac-test":

```bash
guaccollect s3 --s3-url https://play.min.io --s3-bucket guac-test
```

To ingest a single file named "alpine-cyclonedx.json" from the bucket in the
previous example:

```bash
guaccollect s3 --s3-url https://play.min.io --s3-bucket guac-test --s3-item alpine-cyclonedx.json
```

## Google Cloud Storage

`guaccollect` supports the Google Cloud Storage (GCS) blob store. To collect
files from a GCS bucket named "my-bucket" with credentials stored in the local
file `/secret/sa.json`:

```bash
guacone collect gcs my-bucket --gcs-credentials-path /secret/sa.json
```
