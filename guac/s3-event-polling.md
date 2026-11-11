---
layout: page
title: S3 event notification polling
permalink: /guac/s3-event-polling/
parent: "Ingesting SBOMs into GUAC"
---

# S3 event notification polling

The S3 collector can process object events delivered through Kafka or SQS.

Configure your S3 bucket to publish notifications to a topic or queue.

## Message-provider options

- `--s3-mp`: Choose `kafka` (default) or `sqs`.
- `--s3-mp-endpoint`: Broker endpoint for Kafka polling.
- `--s3-queues`: Comma-separated queue or topic names.
- `--s3-bucket`: S3 bucket name; required for collection.
- `--s3-region`: AWS region; defaults to `us-east-1`.

The `--s3-queues` option is required for event polling.

The `--s3-mp-endpoint` option is required when using Kafka.

For SQS, the message-provider endpoint can be omitted.

## Kafka polling example

Use `--poll` with the `guacone collect s3` command:

```bash
guacone collect s3 \
  --s3-url http://localhost:9000 \
  --s3-bucket guac-test \
  --poll \
  --s3-mp kafka \
  --s3-mp-endpoint localhost:9092 \
  --s3-queues sboms
```

The example assumes a reachable S3-compatible store and Kafka broker.

Bucket notifications must already be configured for the `sboms` topic.

## SQS polling example

Select `sqs` and provide the SQS queue name:

```bash
guacone collect s3 \
  --s3-bucket guac-test \
  --poll \
  --s3-mp sqs \
  --s3-queues guac-sboms
```

AWS credentials and the correct AWS region must be available.

The SQS provider resolves the queue name to its queue URL.

The current SQS handler recognizes `ObjectCreated:Put` events.

## Using guaccollect

The `guaccollect s3` command uses `--service-poll` for this mode.

Its `--service-poll` default is `true`.

The message-provider options have the same meanings.

For one-time bucket or object collection, disable event polling.
