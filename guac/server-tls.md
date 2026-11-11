---
layout: page
title: Server-side TLS configuration
permalink: /guac/server-tls/
parent: How GUAC works
---

# Server-side TLS configuration

GUAC supports server-side TLS for CollectSub, GraphQL, and REST.

Each server requires a PEM-encoded certificate and private key.

## TLS flags

- **CollectSub:** `--csub-tls-cert-file` and `--csub-tls-key-file`.
- **GraphQL:** `--gql-tls-cert-file` and `--gql-tls-key-file`.
- **REST:** `--rest-api-tls-cert-file` and `--rest-api-tls-key-file`.

The default port for CollectSub is `2782`.

The default port for GraphQL is `8080`.

The default port for REST is `8081`.

All six certificate and key options default to empty strings.

A server enables TLS only when both file paths are provided.

If one path is missing, the server starts without TLS.

Protect private-key files with suitable filesystem permissions.

## Enable TLS for CollectSub

```bash
guaccsub \
  --csub-tls-cert-file /etc/guac/tls.crt \
  --csub-tls-key-file /etc/guac/tls.key
```

Use the equivalent `gql-*` flags to enable HTTPS for GraphQL.

Use the `rest-api-*` flags to enable HTTPS for REST.

## Connect a CollectSub client

Enable `--csub-tls` when the server requires TLS.

Ensure the client trusts the server certificate.

The option `--csub-tls-skip-verify` disables verification.

Do not use it for production connections.
