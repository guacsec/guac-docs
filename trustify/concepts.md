---
title: Trustify Concepts
layout: page
permalink: /trustify/concepts/
nav_order: 1
parent: Trustify Docs
has_children: true
---

# Trustify Concepts

The following sections explain a few concepts of Trustify.

## Advisory

An advisory is an opinion about a vulnerability.

These opinions include the context to which the opinions apply. These opinions
include evaluation of the severity and scoring of a vulnerability within that
context, such as Common Vulnerability Scoring System (CVSS) scores.

As mentioned in [Vulnerability](#vulnerability), a CVE Record from the CVE
Project is a low-value advisory that mentions the vulnerability and provides a
base opinion about it. It might include CVSS scores, within the context of the
abstract origin containing the vulnerability. This might be simply in reference
to the vulnerability as it exists in source-code form.

Other, more-involved stakeholders, such as, product vendors, upstream project
owners, might issue additional advisories. These opinions might be in reference
to concrete, shipped products, contextualized to how the vulnerable code is
actually used.

## Artifact

For a given package, there can be zero or more instances of that package. Given
`log4j-1.2.3.jar`, seventeen different people could compile the same source with
the same arguments, and still end up with 17 distinct Java jars, due to
non-reproducible builds. Each is an artifact of the same package. Each might,
and most likely will, have its own SHA-256 related to it.

Consider an artifact to be a concrete instance of a package.

## Package

A package is an atomic artifact or component. Packages can be addressed using
pURLs. A package can be described by an SBOM describing how it is created and
its contents. A package can certainly contain other packages. For example,
shading one Java jar into another. A package can also be the sole member of a
product. For example `UBI-8.0.13-x86.oci` can be the singular package within the
"UBI 8.0.13-x86" product. A package is one step more abstract than an artifact.

### pURL

Package URLs (pURLs) are possibly ambiguous names applied to packages. A simple
pURL such as `pkg:maven/org.apache/log4j@1.2.3` can or cannot refer to a unique
artifact. With additional qualifiers, it is possible to produce a URI that
asserts uniqueness, such as
`pkg:maven/org.apache/log4j@1.2.3?repository_url=repo.jboss.com`. Without
additional qualifiers, the implicit aspects, such as `repository_url`, must be
taken into account. For instance, an unqualified `pkg:maven` pURL implies "the
jar from Maven Central, and none other".

## Product

A product is a named collection of 1 or more packages for a concrete shippable
thing.

Products can be addressed using Common Platform Enumerations (CPE) or some other
future identification method. A product can be described by an SBOM describing
its components, which might be other products or packages, or their SBOMs.

`Red Hat Enterprise Linux 8` might be a product stream.
`Red Hat Enterprise Linux 8.2.03 PowerPC` might be a concrete product distinct
from `Red Hat Enterprise Linux 8.2.03 AArch64`.

### CPE

A CPE is a "Common Product Enumeration" from the NIST organization. CPEs are
self-assigned but registered occasionally with NIST. CPEs describe the vendor,
the product, the version, target architecture, etc. CPEs can also be non-fully
specified, to use as pattern-matching. For instance, "All versions of Red Hat
Enterprise Linux 8.2.013, regardless of platform", or if more fully-specified,
could imply "All versions of Red Hat Enterprise Linux 8.x on AArch64".

{: .note } CPEs are somewhat contentious, and used enough for us to not ignore,
but not used enough to be a pivotal definition of "product" for any users of
Trustify.

## SBOM

An SBOM is a source-of-someone’s-truth about "what’s inside it?", so everything
in our DB is ultimately sourced from some source-of-truth. We cannot really say
definitively "product X is composed of A1, A2 + A3". Instead, we can have
multiple simultaneous statements — SBOM’s — from multiple people saying "product
X is claimed by Bob to be A1 + A2" and "product X is claimed by Jim to be A1 +
A97". So an SBOM is the entity to track the origin of the supposed "evidence" of
assertional statements about products…​ about packages…​ about vulnerabilities…​

## Vulnerability

A vulnerability is primarily a name for ensuring all advisories are discussing
the same thing. Generally, most vulnerabilities come from the CVE Project, with
the format of CVE-2024-1234.

Within the database, Trustify adds the vulnerability when discovering an
advisory mentioning it.

A CVE Record from National Institute of Standards and Technology (NIST)/National
Vulnerabilities Database (NVD) is a low-value advisory that is generally the
first discovered advisory that mentions a vulnerability.
