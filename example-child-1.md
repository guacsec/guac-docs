---
layout: page
title: How GUAC components work together
permalink: /guac-components/
parent: Example parent page
nav_order: 1
---

# How GUAC components work together
{: .no_toc }

<details open markdown="block">
  <summary>
    Table of contents
  </summary>
  {: .text-delta }
- TOC
{:toc}
</details>

The full GUAC component deployment is a set of asynchronous services that
combine to form a robust and scaleable pipeline. This is represented by the area
in green in the diagrom below. In some of our [demos](demos/), you may have seen
these components work in concert! This document explains a little more of what
goes on behind the hood!

## GUAC Components

![Guac Diagram](GUAC-diagram.svg)

### GraphQL Server

The GraphQL server serves the GUAC defined nodes through GraphQL queries. It is
an abstraction layer for GUAC integrations and other GUAC components. Our setup docs
use the built-in in-memory backend for the server. Currently the
server also supports a Neo4j backend. Any future backend database support added
to GUAC will not affect the GraphQL interface that the server provides.

### Ingestion Pipeline

| Name       | Short Description                                                                  |
| ---------- | ---------------------------------------------------------------------------------- |
| Collector  | Reads or watches locations for new documents and collects them when found          |
| Ingestor   | Takes documents (ex: SBOMs) and parses them into the GUAC data model/ontology      |
| Assembler  | Takes GUAC objects and puts them in a datastore queryable by the GraphQL server    |
| CollectSub | Takes identifiers of interest, and creates a subscription for collectors to follow |

#### Collectors 

Collectors are meant to gather data from various sources, both internally within
the organization, public sources (like open source), as well as third party
vendors. There are different collectors for the different types of locations
that GUAC can watch: files, storage buckets, git repositories, etc.

Collectors are able to be configured to use a CollectSub service to know which
data sources are of interest. For example, a git collector may subscribe to the
CollectSub service to know which git repositories it should get its data from.
This is a way in which one can, at a glance get an idea of what data sources the
instance of GUAC is looking at!

Collectors and certifiers then take the documents and pass them on to the
ingestor via [Nats](https://nats.io/),

##### Certifiers

Another class of GUAC collector components are certifiers. Certifiers run
outside the server, but are not part of the ingestion pipeline. They watch the
server for new nodes in the server, and then try to add additional information
attached to those nodes.

For example, the [OSV](https://ossf.github.io/osv-schema/) certifier will watch
GUAC for new packages, then try to discover OSV vulnerabilities for those
packages. If any vulnerabilities are found, the certifier will attach a
"CertifyVuln" node to the package that signifies that the package is connected
to the OSV vulnerability.

#### Ingestor

Ingestors take in documents and parse them into the GUAC data model/ontology.
The process extracts meaning from documents and translates to a common reasoning
model (GUAC ontology). In the process, it also finds identifiers of interest in
which it passes to the CollectSub service to request additional information for.

Today, GUAC can understand multiple data formats such as: SPDX, CycloneDX, SLSA,
etc. The ingestor listens for documents to parse via [Nats](https://nats.io/),
and talks to the Assembler via a GraphQL API.

#### Assembler

The assembler takes the parsed GUAC ontology objects from the Ingestor and
creates entries within a database which is used as a source of truth for GUAC
queries.

The assembler exposes a set of GraphQL mutate interfaces (and is physically part
of the GraphQL server, but logically part of ingestion).

#### CollectSub

The collect subcriber service provides a way to express a want for a datasource
to be used, or indication that more data about a software identifier is desired.
For example, in parsing an SBOM, if it sees the use of a package with a PURL,
the ingestor creates and entry to the CollectSub service to indicate more
information about the PURL is desired (via gRPC).

The collectors all subscribe to this service and will automatically retrieve
more information about the PURL (or other identifier/datasource) if it knows how
to handle it. For example, the deps.dev collector would know how to handle
PURLs, and then retrieve more information about the PURL entries created from
the ingestor parsing the SBOM.
