---
layout: page
title: CollectSub gRPC health checks and reflection
permalink: /guac/collectsub-grpc-health/
parent: How GUAC components work together
---

# CollectSub gRPC health checks and reflection

The `guaccsub` server supports gRPC health checks and reflection.

The default listening port is `2782`.

Use `--csub-listen-port` to configure another port.

## Check health with grpcurl

Check the overall status of a server without TLS:

```bash
grpcurl -plaintext -d '{"service":""}' localhost:2782 grpc.health.v1.Health/Check
```

A healthy running server reports `"status": "SERVING"`.

Query the named CollectSub service:

```bash
grpcurl -plaintext -d '{"service":"guacsec.guac.collect_subscriber.schema.CollectSubscriberService"}' localhost:2782 grpc.health.v1.Health/Check
```

The registered service name is:

```text
guacsec.guac.collect_subscriber.schema.CollectSubscriberService
```

This health status does not validate downstream GUAC services.

## Inspect the API with reflection

Reflection enables service discovery without local proto files:

```bash
grpcurl -plaintext localhost:2782 list
grpcurl -plaintext localhost:2782 describe guacsec.guac.collect_subscriber.schema.CollectSubscriberService
```

Use `-plaintext` only when TLS is disabled.

For TLS connections, configure the client to trust the server.

The server TLS flags are `--csub-tls-cert-file` and `--csub-tls-key-file`.

## Kubernetes readiness probe

Kubernetes supports gRPC readiness checks.

For a plaintext CollectSub server on port `2782`, use:

```yaml
readinessProbe:
  grpc:
    port: 2782
    service: guacsec.guac.collect_subscriber.schema.CollectSubscriberService
  initialDelaySeconds: 5
  periodSeconds: 10
```

gRPC probes require a numeric port and use plaintext by default.

TLS mode needs a compatible Kubernetes version and feature gate.

In Kubernetes v1.37, gRPC probe TLS is an alpha feature.

A successful check is not an end-to-end GUAC pipeline test.
