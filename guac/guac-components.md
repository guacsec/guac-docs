---
layout: page
title: How GUAC components work together
permalink: /guac/guac-components/
redirect_from: /guac-components/
parent: How GUAC works
has_children: yes
nav_order: 2
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
combine to form a robust and scalable pipeline. In some of our [demos]({{
site.baseurl }}{guac/guac-use-cases.md %}), you may have seen these components
work in concert. Read on to learn more of what goes on behind the hood!

## GUAC Components

![Guac Diagram](../assets/images/GUACcomponentsdiagram.svg)

### GraphQL Server

The GraphQL server serves the GUAC defined nodes through GraphQL queries. It is
an abstraction layer for GUAC integrations and other GUAC components. Our setup
docs use the built-in, in-memory backend for the server. For persistent storage,
the server supports a Postgresql backend using [ent](https://entgo.io/).

### Ingestion Pipeline

| Name       | Short Description                                                                 |
| ---------- | --------------------------------------------------------------------------------- |
| Collector  | Reads or watches locations for new documents and collects them when found         |
| Ingestor   | Takes documents (ex: SBOMs) and parses them into the GUAC data model/ontology     |
| Assembler  | Takes GUAC objects and puts them in a datastore, queryable by the GraphQL server  |
| CollectSub | Takes identifiers of interest and creates a subscription for collectors to follow |

#### Collectors

Collectors gather data from various sources. These sources can be internal to
the organization, public (like open source), or third-party vendors. There are
different collectors for the different types of locations that GUAC can watch
(files, storage buckets, Git repositories, etc.).

Collectors can be configured to use a CollectSub service to know which data
sources are of interest. For example, a Git collector may subscribe to the
CollectSub service to know which Git repositories it should get its data from.
This is a way to get an idea of what data sources the instance of GUAC is
looking at, at a glance.

Collectors and certifiers then take the documents and pass them onto the
ingestor via [NATS](https://nats.io/).

The following collectors are considered stable:

- filesystem

Other collectors are considered experimental and subject to behavior changes.

##### Certifiers

Another class of GUAC collector components are certifiers. Certifiers run
outside the server, but are not part of the ingestion pipeline. They watch the
server for new nodes in the server and add additional information attached to
those nodes.

For example, the [OSV](https://ossf.github.io/osv-schema/) certifier will watch
GUAC for new packages, then try to discover OSV vulnerabilities for those
packages. If any vulnerabilities are found, the certifier will attach a
"CertifyVuln" node to the package to signify that the package is connected to
the OSV vulnerability.

#### Ingestor

Ingestors take in documents and parse them into the [GUAC data
model/ontology]({{ site.baseurl }}{%link guac/guac-ontology.md %}). This process
extracts meaning from documents and translates them into a common reasoning
model (GUAC ontology). In the process, it also finds identifiers of interest
that it passes to the CollectSub service to request additional information for.

Today, GUAC can understand multiple data formats like SPDX, CycloneDX, and SLSA.
The ingestor listens for documents to parse via [NATS](https://nats.io/), and
talks to the assembler via a GraphQL API.

The ingestion of the following documents is considered stable:

- [Common Security Advisory Framework](https://www.csaf.io/) (CSAF)
- [OpenVEX](https://github.com/openvex)
- [CycloneDX](https://cyclonedx.org/specification/overview/)
- [Dead Simple Signing Envelope](https://github.com/secure-systems-lab/dsse)
  (DSSE)
- [in-toto ITE6](https://github.com/in-toto/ITE/blob/master/ITE/6/README.adoc)
- [SPDX](https://spdx.dev/use/specifications/)
- [OpenSSF Scorecard](https://scorecard.dev/)

Document ingestion from the following sources is considered stable:

- pubsub based on gocloud.dev SDK or NATS
- Supported blobstores based on gocloud.dev SDK
  - azure
  - Google Cloud Storage
  - Amazon S3
  - Memblob
  - Regular file system blobs

Graph enrichment from the following sources is considered stable:

- OSV for vulnerabilities
- ClearlyDefined for license information
- Deps.dev for dependency information

#### Assembler

The assembler takes the parsed GUAC ontology objects from the ingestor and
creates entries within a database. This database is used as a source of truth
for GUAC queries.

The assembler exposes a set of GraphQL mutate interfaces. The assembler is
physically part of the GraphQL server, but logically part of ingestion.

#### CollectSub

The collect subscriber service provides a way to ask for a datasource to be used
or indicate that more data about a software identifier is desired. For example,
in parsing an SBOM, if it sees the use of a package with a pURL, the ingestor
creates an entry to the CollectSub service to indicate more information about
the pURL is desired (via gRPC).

The collectors all subscribe to this service and will automatically retrieve
more information about the pURL (or other identifier/datasource) if they know
how to handle it. For example, the deps.dev collector knows how to handle pURLs
and retrieves more information about the pURL entries created from the ingestor
parsing the SBOM.
