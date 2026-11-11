---
layout: page
title: Annotating metadata with guacone
permalink: /guac/annotating-metadata/
parent: How GUAC components work together
nav_order: 2
---

# Annotating metadata with guacone

The `guacone annotate-metadata` command adds a `HasMetadata` key-value entry for
a package, source, or artifact in the GUAC graph. It sends the annotation to a
running GUAC GraphQL service; it does not collect documents.

## Command syntax

```bash
guacone annotate-metadata [flags] <type> <subject> <key> <value>
```

The four positional arguments identify what to annotate:

- `type` is `package`, `source`, or `artifact`.
- `subject` identifies the item, using the format appropriate to its type:
  - **Package:** A package URL (PURL), such as
    `pkg:golang/github.com/guacsec/guac@v0.0.0`.
  - **Source:** A VCS identifier, such as `git+https://github.com/guacsec/guac`.
  - **Artifact:** An `algorithm:digest` pair, such as a SHA-256 digest.
- `key` is the metadata field name.
- `value` is the value to store for that field.

For example, annotate a package with an owner and a justification:

```bash
guacone annotate-metadata --justification "Reviewed by security team" \
  package "pkg:golang/github.com/guacsec/guac@v0.0.0" owner security
```

## Useful flags

- `--justification` records why the annotation was added. Without this flag,
  GUAC uses `Added by user via guacone`.
- `--gql-addr` selects the GraphQL endpoint (by default,
  `http://localhost:8080/query`).
- `--header-file` supplies HTTP request headers for the GraphQL connection.
- `--package-name` matches all versions of a package instead of only the
  specified version; it applies when the subject type is `package`.

The command requires a reachable GUAC GraphQL service. For metadata labels added
as documents are collected, see the separate `guaccollect --label` option;
`annotate-metadata` adds metadata manually through `guacone`.
