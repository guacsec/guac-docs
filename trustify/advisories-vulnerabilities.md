---
title: Advisories & Vulnerabilities
layout: home
permalink: /trustify/advisories-vulnerabilities/
nav_order: 1
parent: Trustify Concepts
---

# Advisories & Vulnerabilities

Trustify learns about vulnerabilities by ingesting advisories. During the
ingestion process, Trustify extracts and aggregates vulnerability information,
grouped by their vulnerability identifier.

Advisories can contain multiple vulnerabilities and can scope the application of
statements the advisories make to certain packages. This means that Trustify has
an aggregated set of information for a vulnerability, where information from the
Common Vulnerabilities and Exposures (CVE) project supersedes information from
more specific advisories.

Trustify also has "vulnerabilities belonging to an advisory", which contain
specific vulnerability information, provided by that advisory.
