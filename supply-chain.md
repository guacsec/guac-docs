---
layout: page
title: Reacting to a supply chain incident
permalink: /supply-chain/
parent: GUAC use cases
nav_order: 1
---

# Reacting to a supply chain incident

A new high-profile vulnerability landed and now you're wondering how you should react to it. 

How do you discover which of your products and software are vulnerable? 
How should you go about remediating the problem in your organization? 
What is the patch plan?

Using the GraphQL API, you can expose the necessary information to discover how your organization's software catalog is affected and remediate against large-scale security incidents.

This demo simulates the discovery of a high-profile vulnerability and
shows how you can discover what software needs to be reviewed or patched. In the future,
CertifyBad/CertifyGood will be similar to a binary authorization,
where certain checks or policies have determined that an artifact should be
utilized or not.

## Requirements

- [Go](https://go.dev/doc/install)
- [Git](https://github.com/git-guides/install-git)
- [npm](https://docs.npmjs.com/downloading-and-installing-node-js-and-npm)
- [Docker](https://docs.docker.com/get-docker/)
- A fresh copy of the [GUAC service infrastructure through Docker Compose](https://guac.sh/setup/). 

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

[Start up the GUAC visualizer](https://guac.sh/guac-visualizer/).

## Step 5: Mark packages as bad when a security incident occurs

A new security incident has occurred and various communities have pointed out that a particular package is affected. In this scenario, the package "maven/org.apache.logging.log4j/log4j-core" has a vulnerability (yikes!).

The first step we can take is to mark this package as bad by using the `guacone certify` command. This command defaults to assert a negative
certification (instead of a positive one), as well as a `justification` to indicate why the package is bad. In this case, it is a critical vulnerability:

```bash
bin/guacone certify package "never use this version of log4j" "pkg:maven/org.apache.logging.log4j/log4j-core@2.8.1"
```

If we successfully added "CertifyBad", the output will show:

```bash
{"level":"info","ts":1683130083.9894989,"caller":"helpers/assembler.go:69","msg":"assembling CertifyBad: 1"}
```

## Step 6: Check if you are affected

To find out if you're affected by the security incident and decide what you need to patch, utilize the [Guac Visualizer](https://guac.sh/guac-visualizer/). The GUAC visualizer provides a utility to do some basic analysis and exploration
of the software supply chain. This is a great way to get a sense of the size of
the problem and helps when developing prototype utilities and queries with GUAC (very much like the [vulnerability CLI](https://guac.sh/querying-via-cli/)).

## Step 7: Explore bad packages

1. To explore all the "certifyBad" items (packages, sources, or
  artifacts), run the "query Bad" CLI:
  ```bash
  ./bin/guacone query bad
  ```

  This query will automatically search the database and find the list of
  "certifyBad" that are present. For example, an output will look like the
  following:

  ```bash
  Use the arrow keys to navigate: ↓ ↑ → ←
  ? Select CertifyBad to Query:
    ▸ pkg:golang/github.com/kr/pretty (pretty bad undisclosed vuln)
      git+https://github.com/googleapis/google-cloud-go (github repo compromised)
      pkg:golang/github.com/pmezard/go-difflib (github repo compromised)
      pkg:maven/org.apache.logging.log4j/log4j-core@2.8.1 (never use this version of log4j)
  ↓   pkg:golang/github.com/prometheus/client_golang@v1.11.1 (undisclosed vuln)
  ```

2. Select a package, source, or artifact from the list to generate a visualizer URL containing all the dependent packages and artifacts
   (packages that use the certifyBad items).

  Further iterations of the same CLI tool (or another) could be used
  to give a step-by-step guide to remediation!

  For this scenario, select the 
  `pkg:maven/org.apache.logging.log4j/log4j-core (never use this version of log4j)`
  that we created earlier:

    ```bash
    Use the arrow keys to navigate: ↓ ↑ → ←
    ? Select CertifyBad to Query:
    ↑   pkg:golang/github.com/kr/pretty (pretty bad undisclosed vuln)
        git+https://github.com/googleapis/google-cloud-go (github repo compromised)
        pkg:golang/github.com/pmezard/go-difflib (github repo compromised)
      ▸ pkg:maven/org.apache.logging.log4j/log4j-core@2.8.1 (never use this version of log4j)
       pkg:golang/github.com/prometheus/client_golang@v1.11.1 (undisclosed vuln)
    ```

   Doing so will produce a output similar to this:

    ```bash
    ✔ pkg:maven/org.apache.logging.log4j/log4j-core (never use this version of log4j)
    Visualizer url: http://localhost:3000/?path=142367,15573,15572,15515,2509,15574,15337,15336,15335,2
    ```

3. Navigate to the URL to visualize the output. 
This will show an expanded graph of dependencies. 

**If you don't see dependencies expanded here, or less than what is shown below**, re-run the CLI in a few minutes. Additional open-source insights might still be being ingested from GUAC's external sources (such as Deps.dev).

![An image of the visualizer output graph](assets/images/supplychain_dependencies_graph.png)

We can tell from this small example (arranging the graph a little) that
the bad package is being used by a test image under the guacsec org! We need to
remediate that right away!

## Running through a more complex example

The above example is a fairly simple one. For this demo, we'll use a git repo that we know 
is producing a bunch of bad packages. We want to mark the repo as compromised, learn which packages are linked to the repo, and figure out where the packages could be used.

For example, let's take the `googleapis/google-cloud-go` git repo. We will begin by certifying it as bad:

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

With this data, we can investigate further and determine which packages are dependent
on these compromised packages and remediate them quickly. 

## Building more advanced patch planning capability

One of the potential next areas of work for the project is to create a CLI to do patch
planning for the organization. Patch planning will allow an organization to determine which packages to update first.  

In order to build more robust patch planning, you can leverage the [GraphQL Query
API](https://guac.sh/graphql/) that GUAC provides. 
