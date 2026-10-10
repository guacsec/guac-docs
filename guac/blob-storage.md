---
layout: page
title: Using blob storage with GUAC
parent: "Ingesting SBOMs into GUAC"
permalink: /guac/ingesting-blob/
redirect_from: /ingesting-blob/
nav_order: 1
---

# Using blob storage with GUAC

GUAC can ingest files from blob storage using
[gocloud/blob](http://gocloud.dev/blob). The collector can download one item
from the storage, all items from a folder, a whole bucket or listen to storage
events using sqs/kafka (poll) and download the files as they are uploaded.

{: .note }

The [GUAC Helm charts](https://github.com/guacsec/helm-charts) include
[MinIO](https://charts.min.io/), an S3-compatible blob store server.

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

GUAC supports Google Cloud Storage (GCS) blob store via both `guacone` and
`guaccollect` CLI commands.

### Using `guacone` CLI

To collect files from a GCS bucket named "my-bucket" with credentials stored in
`/secret/sa.json`:

```bash
guacone collect gcs my-bucket --gcp-credentials-path /secret/sa.json
```

Alternatively, set the `GOOGLE_APPLICATION_CREDENTIALS` environment variable:

```bash
export GOOGLE_APPLICATION_CREDENTIALS=/secret/sa.json
guacone collect gcs my-bucket
```

### Using `guaccollect` CLI

To collect files using `guaccollect`:

```bash
guaccollect gcs my-bucket --gcp-credentials-path /secret/sa.json
```

Alternatively, set the `GOOGLE_APPLICATION_CREDENTIALS` environment variable:

```bash
export GOOGLE_APPLICATION_CREDENTIALS=/secret/sa.json
guaccollect gcs my-bucket
```

## Cloud-agnostic blob collector

The `guaccollect blob` command accepts a single Go Cloud storage URL and
collects documents from S3, GCS, Azure Blob Storage, or a local filesystem. It
is a cloud-agnostic alternative to the provider-specific commands above:

```bash
guaccollect blob "s3://my-bucket?region=us-east-1"
guaccollect blob "gs://my-bucket"
guaccollect blob "azblob://my-container"
guaccollect blob "file:///path/to/sboms"
```

Use the cloud provider's supported environment-based authentication to access
the store. See the [Go Cloud blob guide](https://gocloud.dev/howto/blob/) for
provider-specific URL and credential configuration.

### Scope collection and cap object size

By default, the blob collector reads every object in the selected store. It does
not select files by document type: every in-scope object is passed on for
ingestion. Use a bucket or prefix that contains documents GUAC can ingest.

- `--blob-prefix` limits collection to keys beginning with the supplied prefix,
  such as `sboms/`.
- `--blob-max-object-size` sets the maximum object size to read, in bytes.
  Larger objects are logged and skipped. A value of `0` uses the collector's
  built-in size limit; it does not disable the limit.

To collect only objects under `sboms/` with a 50 MiB per-object cap:

```bash
guaccollect blob --blob-prefix sboms/ --blob-max-object-size 52428800 \
  "s3://my-bucket?region=us-east-1"
```

To periodically poll the store, use `--service-poll` and an interval:

```bash
guaccollect blob --service-poll --interval 5m \
  "s3://my-bucket?region=us-east-1"
```
