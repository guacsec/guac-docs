---
layout: page
title: What is Known and Unknown about your Software Supply Chain?
permalink: /known-and-unknown/
parent: GUAC demos
grand_parent: Getting started with GUAC
nav_order: 1
---

# What is Known and Unknown about your Software Supply Chain

![Dinosaur Meme](assets/images/knownunknownmeme.jpeg)

The software supply chain is like a rabbit hole that can be very deep and hold
many twists and turns. It's hard to track where the different tunnels may lead.
With SBOMs, SLSA attestations, scorecards, VEX, and other in-toto ITE-6
attestations, the list of metadata associated with an artifact is growing. You
may wonder:

Where do I store my SBOMs, SLSA attestations, and other metadata documents? Can
I find these documents quickly? What do I know (and not know) about my supply
chain?

GUAC has the ability to ingest and link these documents together into a complete
picture of your software supply chain. With GUAC you can easily determine your:

- known, knowns
- known, unknowns
- unknown, unknowns

In the ["Expanding your view of the software supply chain" demo]({{
site.baseurl }}{%link expanding-your-view.md %}), we went through the process of
ingesting an SBOM and letting GUAC expand our horizons on what we know about our
environment autonomously. In this demo, we'll take that information and use it to
determine what we know and don't know about the artifacts.

## Requirements

- [Go](https://go.dev/doc/install)
- [Docker](https://docs.docker.com/get-docker/)
- A fresh copy of the [GUAC service infrastructure through Docker
  Compose]({{ site.baseurl }}{%link setup.md %})
- Completion of the [Expanding your view of the software supply chain
  demo]({{ site.baseurl }}{%link expanding-your-view.md %})

## Understanding the data

GUAC, at the time of the beta release, can store various metadata about an
artifact. In the context of query CLI that we will be using, the evidence nodes
have the following definitions:

| Evidence Nodes | Description                                                                  |
| -------------- | ---------------------------------------------------------------------------- |
| `hashEquals`   | When two artifacts are equal                                                 |
| `scorecards`   | The OpenSSF scorecard associated with the source repo                        |
| `occurrences`  | A package is associated with an artifact (digest) (or vice-versa)            |
| `hasSrcAt`     | A package has a source repo at the following location                        |
| `hasSBOMs`     | A package/artifact has an SBOM stored in a downloadable location             |
| `hasSLSAs`     | The artifact has an SLSA attestation stored in a downloadable location       |
| `certifyVulns` | The package has been scanned (currently via OSV) and the results of the scan |
| `vexLinks`     | A VEX document associated with the Vulnerability                             |
| `badLinks`     | List of CertifyBad associated with the package, source or artifact           |
| `goodLinks`    | List of CertifyGood associated with the package, source or artifact          |
| `pkgEquals`    | Two packages (with different purls) are equal                                |

For more information on these, refer to the [grapQL documentation]({{
site.baseurl }}{%link graphql.md %}) and [ontology definitions]({{ site.baseurl
}}{%link guac-ontology-definition.md %}).

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

Build the GUAC binaries using the `make` command.

```bash
make
```

## Step 3. Run the query to find the knowns and unknowns

Utilizing the CLI and GUAC Visualizer, we can determine:

- The location of SBOMs, SLSA attestations, and scorecard information
- What information we are missing

We will utilize the “query known” CLI. This CLI has the ability to search a
package via [PURL](https://github.com/package-url/purl-spec), source URL
following the definition of VCS uri from the
[SPDX documentation](https://spdx.github.io/spdx-spec/v2.3/package-information/#771-description)
`<vcs_tool>+<transport>://<host_name>[/<path_to_repository>][@<revision_tag_or_branch>][#<sub_path>])`
and a artifact (algorithm:digest).

1. Look at if a package
   ([vault](https://github.com/guacsec/guac-data/blob/main/top-dh-sboms/vault.json))
   to see if it has an SBOM associated with and where it can be found:

   Ingest the vault's SBOM:

   ```bash
   ./bin/guacone collect files ../guac-data/top-dh-sboms/vault.json
   ```

   The output should be similar to:

   ```bash
   {"level":"info","ts":1684774157.098919,"caller":"cmd/files.go:167","msg":"[2.054952875s] completed doc {Collector:FileCollector Source:file:///../guac-data/top-dh-sboms/vault.json}"}
   {"level":"info","ts":1684774157.098937,"caller":"cmd/files.go:174","msg":"collector ended gracefully"}
   {"level":"info","ts":1684774157.09894,"caller":"cmd/files.go:187","msg":"completed ingesting 1 documents of 1"}
   ```

   Next Run the query command:

   ```bash
   ./bin/guacone query known package "pkg:guac/spdx/docker.io/library/vault-latest"
   ```

   The output will look similar to this:

   ```bash
   +------------------------------------------------+
   | Package Name Nodes                             |
   +-----------+-----------+------------------------+
   | NODE TYPE | NODE ID   | ADDITIONAL INFORMATION |
   +-----------+-----------+------------------------+
   +-----------+-----------+------------------------+
   Visualizer url: http://localhost:3000/?path=4,3,2
   +----------------------------------------------------------------------------------------------+
   | Package Version Nodes                                                                        |
   +-----------+-----------+----------------------------------------------------------------------+
   | NODE TYPE | NODE ID   | ADDITIONAL INFORMATION                                               |
   +-----------+-----------+----------------------------------------------------------------------+
   | hasSBOM   | 6964      | SBOM Download Location: file:///../guac-data/top-dh-sboms/vault.json |
   +-----------+-----------+----------------------------------------------------------------------+
   Visualizer url: http://localhost:3000/?path=5,4,3,2,6964
   ```

   The output has two separate tables: one for the “package name level” and the
   other at the “package version level”. Evidence/metadata nodes at the “package
   name level” apply to all the versions that come below it. “Package version
   level”, on the other hand, only applies to the version specified. By default,
   if a version is not specified during ingestion, it defaults to an empty
   version string.

   The “package name level” does not have any nodes associated with it, but the
   “package version level” does have the “hasSBOM” node associated with it. This
   shows us that this package has an SBOM associated with it and can be
   downloaded at the following location. Future plans for GUAC are to have an
   evidence store with GUAC to store SBOM and SLSA attestations for quick
   access. For now, we can quickly locate the SBOM. We also learned that we are
   missing `hasSLSA` attestations for this package.

2. Determine if an artifact has an SLSA attestation associated with it

   Along the same lines, we have the hash of an executable binary that was
   generated. We can ask the question, "Is there an SLSA attestation
   associated?" with it. If so, where is it located and is there a package
   associated with this particular hash?

   This time, we will change query for known `artifact` rather than package:

   ```bash
   ./bin/guacone query known artifact "sha256:625fe537a4c1657bd613be44f7882a8883c13c3b72919cfdbd02d2eb4dbf677b"
   ```

   The output should look similar to the following:

   ```bash
   +------------+-----------+-----------------------------------------------------------------------------------------------------------------+
   | NODE TYPE  | NODE ID   | ADDITIONAL INFORMATION                                                                                          |
   +------------+-----------+-----------------------------------------------------------------------------------------------------------------+
   | occurrence | 5477      | Occurrence for Package: pkg:guac/generic/gs%3A/kubernetes-release/release/v1.24.1/bin/windows/arm64/kubectl.exe |
   +------------+-----------+-----------------------------------------------------------------------------------------------------------------+
   | hasSLSA    | 6364      | SLSA Attestation Location: file:///../guac-data/docs/slsa/kube-slsa-v1.24.1.json                                |
   +------------+-----------+-----------------------------------------------------------------------------------------------------------------+
   Visualizer url: http://localhost:3000/?path=5476,5477,6364
   ```

   From the output, we know that there is a SLSA attestation associated with
   this particular artifact and its location. We also see from the output, that
   the digest is associated with the Windows ARM64 version of the kubectl!

   **Note**: Currently, the package "purl" is guessed based on heuristics. This
   will improve as future versions of SLSA adopt
   [ResourceDescriptor field type](https://github.com/in-toto/attestation/blob/main/spec/v1/resource_descriptor.md).

   For more information on the SLSA attestation, we can look up the SLSA
   attestation via the Node ID in the [graphQL
   playground]({{ site.baseurl }}{%link guac-graphql.md %}) that is shown in the
   output.

   ```graphql
   query SLSAQ3 {
     HasSLSA(hasSLSASpec: { id: "6364" }) {
       ...allHasSLSATree
     }
   }
   ```

   To view the output:

   ```graphql
   {
    "data": {
      "HasSLSA": [
        {
          "id": "6364",
          "subject": {
            "id": "5476",
            "algorithm": "sha256",
            "digest": "625fe537a4c1657bd613be44f7882a8883c13c3b72919cfdbd02d2eb4dbf677b"
          },
          "slsa": {
            "builtFrom": [
              {
                "id": "6354",
                "algorithm": "sha1",
                "digest": "aaf024b5e8dc5e08e4414583203968ca0a5ec043"
              }
            ],
            "builtBy": {
              "id": "6356",
              "uri": "https://git.k8s.io/release/docs/krel"
            },
            "buildType": "https://cloudbuild.googleapis.com/CloudBuildYaml@v1",
            "slsaPredicate": [
              {
                "key": "slsa.buildType",
                "value": "https://cloudbuild.googleapis.com/CloudBuildYaml@v1"
              },
              {
                "key": "slsa.builder.id",
                "value": "https://git.k8s.io/release/docs/krel"
              },
              {
                "key": "slsa.invocation.configSource.entryPoint",
                "value": "https://git.k8s.io/release/gcb/stage/cloudbuild.yaml"
              },
              {
                "key": "slsa.invocation.parameters.branch",
                "value": "release-1.24"
              },
              {
                "key": "slsa.invocation.parameters.build-version",
                "value": "v1.24.1-rc.0.29+1868a3a2eb9945"
              },
              {
                "key": "slsa.invocation.parameters.nomock",
                "value": "true"
              },
              {
                "key": "slsa.invocation.parameters.type",
                "value": "official"
              },
              {
                "key": "slsa.materials.0.digest.sha1",
                "value": "'aaf024b5e8dc5e08e4414583203968ca0a5ec043'"
              },
              {
                "key": "slsa.materials.0.uri",
                "value": "git+https://github.com/kubernetes/kubernetes"
              },
              {
                "key": "slsa.metadata.buildFinishedOn",
                "value": "2022-05-24T13:27:46.145159431Z"
              },
              {
                "key": "slsa.metadata.buildInvocationID",
                "value": "b6186090-c8ff-4f91-97cf-7a3b47699e57"
              },
              {
                "key": "slsa.metadata.buildStartedOn",
                "value": "2022-05-24T12:13:35.054695403Z"
              },
              {
                "key": "slsa.metadata.completeness.environment",
                "value": "false"
              },
              {
                "key": "slsa.metadata.completeness.materials",
                "value": "true"
              },
              {
                "key": "slsa.metadata.completeness.parameters",
                "value": "true"
              },
              {
                "key": "slsa.metadata.reproducible",
                "value": "false"
              }
            ],
            "slsaVersion": "https://slsa.dev/provenance/v0.2",
            "startedOn": "2022-05-24T12:13:35.054695403Z",
            "finishedOn": "2022-05-24T12:13:35.054695403Z",
            "origin": "file:///../guac-data/docs/slsa/kube-slsa-v1.24.1.json",
            "collector": "FileCollector"
          }
        }
      ]
    }
   }
   ```

3. Run the query on the Prometheus package we were working on within the
   workflow demo:

   ```bash
   ./bin/guacone query known package "pkg:golang/github.com/prometheus/client_golang@v1.11.1"
   ```

   The output should be similar to this:

   ```bash
   +---------------------------------------------------------------------------------+
   | Package Name Nodes                                                              |
   +-----------+-----------+---------------------------------------------------------+
   | NODE TYPE | NODE ID   | ADDITIONAL INFORMATION                                  |
   +-----------+-----------+---------------------------------------------------------+
   | hasSrcAt  | 7647      | Source: git+https://github.com/prometheus/client_golang |
   +-----------+-----------+---------------------------------------------------------+
   Visualizer url: http://localhost:3000/?path=578,327,6,7647
   +----------------------------------------------------+
   | Package Version Nodes                              |
   +-------------+-----------+--------------------------+
   | NODE TYPE   | NODE ID   | ADDITIONAL INFORMATION   |
   +-------------+-----------+--------------------------+
   | certifyVuln | 13469     | vulnerability ID: NoVuln |
   +-------------+-----------+--------------------------+
   Visualizer url: http://localhost:3000/?path=579,578,327,6,13469

   ```

   In this example, the “package name level” has the “hasSrcAt” node that shows
   us that the “prometheus/client_golang” source repo is located at
   “<https://github.com/prometheus/client_golang>”.

   The “Package Version Nodes” shows us that the specific package with the purl
   `pkg:golang/github.com/prometheus/client_golang@v1.11.1` was scanned and did
   not contain any vulnerabilities associated with it.

   **NOTE**: This is just the vulnerability associated with this specific
   package (not taking into account dependencies). For a full in-depth
   vulnerability search please follow the [Query Vulnerability
   demo]({{ site.baseurl }}{%link querying-via-cli.md %}).

   We also see that in this case, we did not get a `hasSBOM` associated with it.
   Meaning that we do not have any SBOM information related to this package. We
   also see there are no nodes for the SLSA attestations, but we could query the
   source for more information about it and its scorecard information.

   But before we do that, we will look at another version of the
   “prometheus/client_golang” package with the purl
   "pkg:golang/github.com/prometheus/client_golang@v1.4.0". Note the version is
   now v1.4.0.

4. Use the CLI to query the other version:

   ```bash
   ./bin/guacone query known package "pkg:golang/github.com/prometheus/client_golang@v1.4.0"
   ```

   ```bash
   +---------------------------------------------------------------------------------+
   | Package Name Nodes                                                              |
   +-----------+-----------+---------------------------------------------------------+
   | NODE TYPE | NODE ID   | ADDITIONAL INFORMATION                                  |
   +-----------+-----------+---------------------------------------------------------+
   | hasSrcAt  | 7647      | Source: git+https://github.com/prometheus/client_golang |
   +-----------+-----------+---------------------------------------------------------+
   Visualizer url: http://localhost:3000/?path=578,327,6,7647
   +-----------------------------------------------------------------+
   | Package Version Nodes                                           |
   +-------------+-----------+---------------------------------------+
   | NODE TYPE   | NODE ID   | ADDITIONAL INFORMATION                |
   +-------------+-----------+---------------------------------------+
   | certifyVuln | 13471     | vulnerability ID: ghsa-cg3q-j54f-5p7p |
   | certifyVuln | 13473     | vulnerability ID: go-2022-0322        |
   +-------------+-----------+---------------------------------------+
   Visualizer url: http://localhost:3000/?path=7600,578,327,6,13471,13473
   ```

   We once again see that at the “package name level” we have the same source
   repo associated with it. But at the “package version level” (at v1.4.0) there
   are some vulnerabilities that are associated with it! The newer version of
   the same “prometheus/client_golang” did not have these vulnerabilities. We
   could do further investigation via the GUAC visualizer (and other upcoming
   tools) to determine which packages are dependent on these and need to be
   updated to a newer version. For example, from the workflow demo, we know that
   github.com/armon/go-metrics version 0.3.10 depends on this package and should
   be immediately updated!

   ```bash
        {
           "id": "7624",
           "justification": "dependency data collected via deps.dev",
           "package": {
             "id": "6",
             "type": "golang",
             "namespaces": [
               {
                 "id": "279",
                 "namespace": "github.com/armon",
                 "names": [
                   {
                     "id": "280",
                     "name": "go-metrics",
                     "versions": [
                       {
                         "id": "281",
                         "version": "v0.3.10",
                         "qualifiers": [],
                         "subpath": ""
                       }
                     ]
                   }
                 ]
               }
             ]
           },
           "dependentPackage": {
             "id": "6",
             "type": "golang",
             "namespaces": [
               {
                 "id": "396",
                 "namespace": "github.com/prometheus",
                 "names": [
                   {
                     "id": "397",
                     "name": "client_golang",
                     "versions": []
                   }
                 ]
               }
             ]
           },
           "versionRange": "v1.4.0",
           "origin": "deps.dev",
           "collector": "deps.dev"
         }
   ```

5. Let's take a closer look at the source repo for the prometheus/client_golang
   package: We can run the query:

   **Note**: `source` is specified to indicate we are querying a source repo.

   ```bash
   ./bin/guacone query known source "git+https://github.com/prometheus/client_golang"
   ```

   ```bash
   +-----------+-----------+--------------------------------------------------------------------+
   | NODE TYPE | NODE ID   | ADDITIONAL INFORMATION                                             |
   +-----------+-----------+--------------------------------------------------------------------+
   | hasSrcAt  | 7647      | Source for Package: pkg:golang/github.com/prometheus/client_golang |
   +-----------+-----------+--------------------------------------------------------------------+
   | scorecard | 7583      | Overall Score: 6.600000                                            |
   +-----------+-----------+--------------------------------------------------------------------+
   Visualizer url: http://localhost:3000/?path=7582,7581,6965,7647,7583
   ```

   From this output, we see the reverse of what we saw when we queried the
   package. This time, we see that this source is related to the
   prometheus/client_golang package we queried for above but also we see that
   there is an OpenSSF scorecard associated.

6. Finally, let’s query for another source repo:

   ```bash
   ./bin/guacone query known source "git+https://github.com/googleapis/google-cloud-go"
   ```

   The output should be similar to:

   ```bash
   +-----------+-----------+---------------------------------------------------------------------+
   | NODE TYPE | NODE ID   | ADDITIONAL INFORMATION                                              |
   +-----------+-----------+---------------------------------------------------------------------+
   | hasSrcAt  | 7074      | Source for Package: pkg:golang/cloud.google.com/go                  |
   | hasSrcAt  | 7075      | Source for Package: pkg:golang/cloud.google.com/go/storage          |
   | hasSrcAt  | 7156      | Source for Package: pkg:golang/cloud.google.com/go/spanner          |
   | hasSrcAt  | 8948      | Source for Package: pkg:golang/cloud.google.com/go/compute/metadata |
   | hasSrcAt  | 8949      | Source for Package: pkg:golang/cloud.google.com/go/logging          |
   | hasSrcAt  | 8950      | Source for Package: pkg:golang/cloud.google.com/go/longrunning      |
   +-----------+-----------+---------------------------------------------------------------------+
   | scorecard | 6968      | Overall Score: 8.300000                                             |
   +-----------+-----------+---------------------------------------------------------------------+
   Visualizer url: http://localhost:3000/?path=6967,6966,6965,7074,7075,7156,8948,8949,8950,6968
   ```

   Here we see that this specific source repo is associated with various
   different packages (go, storage, spanner…etc), and again we see that it does
   have a scorecard associated with a score of 8.3

## Knowing the unknown

Based on the information gathered above:

- We know what we have about the various artifacts (either ingested or determine
  by the services of GUAC).
- We can quickly locate an SBOM, an SLSA attestation (other ITE-6 attestations),
  OpenSSF scorecard information, and other metadata quickly.
- We determined what we don’t know. For example, some packages did not have an
  SBOM or SLSA attestation associated. There may have been source repositories
  that weren't scanned by OpenSSF Scorecard.
- We found information that we didn’t even know that we needed to know. For
  example, the vulnerable version of prometheus/client_golang (the older
  version) is used in our software supply chain and needs to be updated
  immediately. Knowing the unknown is the first key step in securing the supply
  chain. If the security teams and developers have no knowledge of these, how
  can we keep shifting left effectively?
