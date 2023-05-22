---
layout: page
title: Reacting to a supply chain incident
permalink: /supply-chain/
parent: GUAC demos
grand_parent: Getting started with GUAC
nav_order: 4
---

# Reacting to a supply chain incident

A new high-profile vulnerability landed and now you're wondering how you should
react to it.

How do you discover which of your products and software are vulnerable? How
should you go about remediating the problem in your organization? What is the
patch plan?

Using the GraphQL API, you can expose the necessary information to discover how
your organization's software catalog is affected and remediate against
large-scale security incidents.

This demo simulates the discovery of a high-profile vulnerability and shows how
you can discover what software needs to be reviewed or patched. In the future,
CertifyBad/CertifyGood will be similar to a binary authorization, where certain
checks or policies have determined that an artifact should be utilized or not.

## Requirements

- [Go](https://go.dev/doc/install)
- [Git](https://git-scm.com/downloads)
- [npm](https://docs.npmjs.com/downloading-and-installing-node-js-and-npm)
- [Docker](https://docs.docker.com/get-docker/)
- A fresh copy of the [GUAC service infrastructure through Docker
  Compose]({{ site.baseurl }}{%link setup.md %}).

## Step 1: Clone GUAC

1. Clone GUAC to a local directory:

   ```bash
   git clone https://github.com/guacsec/guac.git
   ```

2. Clone GUAC data (this is used as test data for this demo):

   ```bash
   git clone https://github.com/guacsec/guac-data.git
   ```

The rest of the demo will assume you are in the GUAC directory

```bash
cd guac
```

## Step 2: Build the GUAC binaries

Build the GUAC binaries using the `make` command:

```bash
make
```

## Step 3: Set up your organization's software catalog

For this demo, we will simulate ingesting an organization's software catalog. To
do this, we will ingest a collection of SBOMs and SLSA attestations into GUAC:

```bash
bin/guacone collect files ../guac-data/docs/
```

Once ingested we will see the following message (the number of documents may
vary):

```bash
{"level":"info","ts":1681864775.1161852,"caller":"cmd/files.go:201","msg":"completed ingesting 67 documents of 67"}
```

## Step 4: Set up the experimental GUAC Visualizer

To find out if you're affected by the security incident and decide what you need
to patch, utilize the [Guac Visualizer]({{ site.baseurl }}{%link
guac-visualizer.md %}). The GUAC visualizer provides a utility to do some basic analysis
and exploration of the software supply chain. This is a great way to get a sense
of the size of the problem and helps when developing prototype utilities and queries
with GUAC (very much like the [vulnerability CLI]({{
site.baseurl }}{%link querying-via-cli.md %})).

## Step 5: Mark packages as bad when a security incident occurs

A new security incident has occurred and various communities have pointed out
that a particular package is affected. In this scenario, the debian package
"tzdata" has been found to have a critical vulnerability (yikes!). Now that we
know the package and the vulnerable version, can we use this information to
quickly find where this package is being used?

The first step we can take is to mark this package as bad by using the
`guacone certify` command. This command defaults to assert a negative
certification (instead of a positive one), as well as a `justification` to
indicate why the package is bad. In this case, it is a critical vulnerability:

```bash
./bin/guacone certify package "compromised version of tzdata" "pkg:deb/debian/tzdata@2021a-1+deb11u5?arch=all&distro=debian-11"
```

If we successfully added "CertifyBad", the output will show:

```bash
{"level":"info","ts":1683130083.9894989,"caller":"helpers/assembler.go:69","msg":"assembling CertifyBad: 1"}
```

## Step 6: Explore bad packages

1. To explore all the "certifyBad" items (packages, sources, or artifacts), run
   the "query Bad" CLI:

   ```bash
   ./bin/guacone query bad
   ```

   This query will automatically search the database and find the list of
   "certifyBad" that are present. For example, an output will look like the
   following:

   ```bash
   Use the arrow keys to navigate: ↓ ↑ → ←
   ? Select CertifyBad to Query:
       pkg:golang/k8s.io/release/images/build/go-runner@%28devel%29 (compromised go-runner)
     ▸ pkg:deb/debian/tzdata@2021a-1+deb11u5 (compromised version of tzdata)
   ```

2. Select a package, source, or artifact from the list to generate a visualizer
   URL containing all the dependent packages and artifacts (packages that use
   the certifyBad items).

   Further iterations of the same CLI tool (or another) could be used to give a
   step-by-step guide to remediation!

   For this scenario, select the
   `pkg:deb/debian/tzdata@2021a-1+deb11u5 (compromised version of tzdata)` that
   we created earlier.

   Doing so will produce a output similar to this:

   ```bash
   ✔ pkg:deb/debian/tzdata@2021a-1+deb11u5 (compromised version of tzdata)
   Visualizer url: http://localhost:3000/?path=142605,44614,1372,1305,1304,127547,127527,127526,36248,2,125455,125358,125357,123291,123287,123286,121220,121216,121215,119149,119145,119144,117075,117074,117073,115010,115006,115005,112939,112935,112934,110299,110283,110282,107515,107453,107452,68077,67990,67989,65923,65745,65744,63678,63674,63673,61607,61603,61602,59536,59532,59531,57463,57461,57460,55393,55390,55389,53320,53319,53318,51236,51113,51112,49048,48779,48778,46714,46713,46712,44615,44610,44609,42533,42528,42527,40466,40462,40461,38397,38393,38392,36252,36250,36249,15392,15337,15336,15335,4155,4125,4124,3,3182,2865,2864,2667,2633,2632,2501,2419,2418,2413,2312,2311,2190,2150,2149,2092,2048,2047,1374,1303,1302
   ```

3. Navigate to the URL to visualize the output. This will show an expanded graph
   of dependencies.

   ![An image of the visualizer output graph](assets/images/supplychain_dependencies_graph.png)

   We can tell from this example (arranging the graph a little) the bad debian
   package (used for timezone information) is commonly used throughout a bunch
   of dependant container images! All are a cause for concern as they are
   notable images for Kubernetes, Redis, Nginx and Python. We need to remediate
   these right away! This allows us to quickly figure out what needs to be
   updated, so that we are not scrambling to first scan and determine where
   `tzdata` might be used.

## Exploring a known bad source repo

In the above example, we looked at a specific package. For this demo, we'll use
a git repo that we know is producing a bunch of bad packages. We want to mark
the repo as compromised, learn which packages are linked to the repo, and figure
out where the packages could be used.

For example, let's take the `googleapis/google-cloud-go` git repo. We will begin
by certifying it as bad:

```bash
bin/guacone certify source "github repo compromised" "git+https://github.com/googleapis/google-cloud-go"
```

You will see an output confirming that it has been added to the database:

```bash
{"level":"info","ts":1683130083.9894989,"caller":"helpers/assembler.go:69","msg":"assembling CertifyBad: 1"}
```

We perform the same actions by running the CLI but this time selecting the new
compromised source repo:

```bash
? Select CertifyBad to Query:
    pkg:golang/github.com/prometheus/client_golang@v1.4.0 (undisclosed vuln)
    pkg:golang/github.com/dougm/pretty (pretty bad undisclosed vuln)
    pkg:golang/github.com/kr/pretty (pretty bad undisclosed vuln)
  ▸ git+https://github.com/googleapis/google-cloud-go (github repo compromised)
↓   pkg:golang/github.com/pmezard/go-difflib (github repo compromised)
```

Selecting the
`gitt+https://github.com/googleapis/google-cloud-go (github repo compromised)`

will output the following (the IDs path could be different):

```bash
✔ git+https://github.com/googleapis/google-cloud-go (github repo compromised)
Visualizer url: http://localhost:3000/?path=130726,1001,1000,97,130727,130629,4501,130728,130632,130729,130635,130730,4611,131477,4930,131478,131469,131468,133884,5380,133898,5188,133918,4502,133976,5425,134985,134417,134986,134542,138434,130615,130614,5435
```

We can now follow the url to see the following graph:

![An image of the visualizer output graph](assets/images/supplychain_dependencies_large_graph.png)

From this view, we can see that this particular repo is being used by a bunch of
packages, specifically:

| Packages                                        |
| ----------------------------------------------- |
| pkg:golang/cloud.google.com/go                  |
| pkg:golang/cloud.google.com/go/bigquery         |
| pkg:golang/cloud.google.com/go/datastore        |
| pkg:golang/cloud.google.com/go/pubsub           |
| pkg:golang/cloud.google.com/go/storage          |
| pkg:golang/cloud.google.com/go/compute          |
| pkg:golang/cloud.google.com/go/compute/metadata |
| pkg:golang/cloud.google.com/go/iam              |
| pkg:golang/cloud.google.com/go/kms              |
| pkg:golang/cloud.google.com/go/monitoring       |
| pkg:golang/cloud.google.com/go/spanner          |
| pkg:golang/cloud.google.com/go/logging          |
| pkg:golang/cloud.google.com/go/longrunning      |

With this data, we can investigate further and determine which packages are
dependent on these compromised packages and remediate them quickly.

## Building more advanced patch planning capability

One of the potential next areas of work for the project is to create a CLI to do
patch planning for the organization. Patch planning will allow an organization
to determine which packages to update first.

In order to build more robust patch planning, you can leverage the [GraphQL
Query API]({{ site.baseurl }}{%link guac-graphql.md %}) that GUAC provides.
