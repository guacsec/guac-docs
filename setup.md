---
layout: page
title: Set up GUAC with Docker Compose
permalink: /setup/
parent: Getting started with GUAC
nav_order: 1
---

# Set up GUAC with Docker Compose

{: .note }

If you’d prefer, you can set up GUAC with Kubernetes with the experimental
[Helm charts provided by Kusari](https://github.com/kusaridev/helm-charts/tree/main/charts/guac).
Note that these helm charts are still experimental and are hosted in a
third-party repo and may not be synchronized with the GUAC repo.

GUAC consists of multiple components. You may have seen a subset of these used
in various [GUAC demos]({{ site.baseurl }}{% link guac-use-cases.md %}). To get
the most value out of GUAC, you’ll need to set up all components. This tutorial
will walk you through how to deploy GUAC, using Docker Compose, so that you get
the full set of components.

If you’re curious about the various GUAC components and what they do, see [How
GUAC components work together]({{ site.baseurl }}{%link guac-components.md %}).

## Setup video

A video format of these setup instructions is available here:

<iframe width="560" height="315" src="https://www.youtube-nocookie.com/embed/3e-Qurgl3Sc?si=N2z7AAUOj1lM1EG-" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" allowfullscreen></iframe>

<details open markdown="block">
  <summary>
    Table of contents
  </summary>
  {: .text-delta }
- TOC
{:toc}
</details>

## Prerequisites

- [Docker](https://docs.docker.com/get-docker/)
- [Docker Compose](https://docs.docker.com/compose/install/)
- [jq](https://stedolan.github.io/jq/download/)
- [cosign](https://docs.sigstore.dev/system_config/installation/)
- [crane](https://github.com/google/go-containerregistry/blob/main/cmd/crane/README.md)

## Step 0: Verify GUAC image via Cosign

1. Based on the
   [latest GUAC release](https://github.com/guacsec/guac/releases/latest),
   validate that the GUAC image is signed and verifiable via cosign by running
   the following command:

```bash
LATEST_VERSION=$(curl https://api.github.com/repos/guacsec/guac/releases/latest | grep tag_name | cut -d : -f2 | tr -d "v\", ")
GUAC_DIGEST=$(crane digest ghcr.io/guacsec/guac:v$LATEST_VERSION)

cosign verify ghcr.io/guacsec/guac@$GUAC_DIGEST \
  --certificate-oidc-issuer https://token.actions.githubusercontent.com  \
  --certificate-identity https://github.com/guacsec/guac/.github/workflows/release.yaml@refs/tags/v$LATEST_VERSION
```

You should see an output similar to this:

```bash
Verification for ghcr.io/guacsec/guac@sha256:de50517b5a527f031395ba11de5576462bc4db6fa0eef5073f82fab052c2b07e --
The following checks were performed on each of these signatures:
  - The cosign claims were validated
  - Existence of the claims in the transparency log was verified offline
  - The code-signing certificate was verified using trusted certificate authority certificates

[{"critical":{"identity":{"docker-reference":"ghcr.io/guacsec/guac"},"image":{"docker-manifest-digest":"sha256:de50517b5a527f031395ba11de5576462bc4db6fa0eef5073f82fab052c2b07e"},"type":"cosign container image signature"},"optional":{"1.3.6.1.4.1.57264.1.1":"https://token.actions.githubusercontent.com","1.3.6.1.4.1.57264.1.2":"push","1.3.6.1.4.1.57264.1.3":"463b8004beebbd413ecf556e4fc5a1bf986534ab","1.3.6.1.4.1.57264.1.4":"release","1.3.6.1.4.1.57264.1.5":"guacsec/guac","1.3.6.1.4.1.57264.1.6":"refs/tags/v0.1.2","Bundle":{"SignedEntryTimestamp":"MEUCIBuRzf/8IPBSjPINRC1XvzmSUhX83wGj+tX+g/7FektaAiEAq84FtWJAj+39qf8AB9ZJvbLOUUGCPdM9SsC1mDyZW24=","Payload":{"body":"eyJhcGlWZXJzaW9uIjoiMC4wLjEiLCJraW5kIjoiaGFzaGVkcmVrb3JkIiwic3BlYyI6eyJkYXRhIjp7Imhhc2giOnsiYWxnb3JpdGhtIjoic2hhMjU2IiwidmFsdWUiOiIzMzBhYjVjNDcwZGU5ZGZlOTJkOGVkMzVjZTJmNDQ0OWMzY2EwMTI0ZWFmOTlkZmExZjE0ZTk2NjE2YWQ1MzJkIn19LCJzaWduYXR1cmUiOnsiY29udGVudCI6Ik1FWUNJUUQzangyVlV2WiszZndpM1VubWlmS1BEckNMOWVuQlJWR0tTNzQrU1YzVWl3SWhBTnpaRkRYL3V4TWtJUjA0TnU1clJyVkYvejFHSUt3dU50UngyUitTWGZkcSIsInB1YmxpY0tleSI6eyJjb250ZW50IjoiTFMwdExTMUNSVWRKVGlCRFJWSlVTVVpKUTBGVVJTMHRMUzB0Q2sxSlNVZHBla05EUW1oTFowRjNTVUpCWjBsVlpGb3dkMWhVVlROcFVEaFVXRmw1VUZBM05IaFVNVVZGZGtaQmQwTm5XVWxMYjFwSmVtb3dSVUYzVFhjS1RucEZWazFDVFVkQk1WVkZRMmhOVFdNeWJHNWpNMUoyWTIxVmRWcEhWakpOVWpSM1NFRlpSRlpSVVVSRmVGWjZZVmRrZW1SSE9YbGFVekZ3WW01U2JBcGpiVEZzV2tkc2FHUkhWWGRJYUdOT1RXcE5kMDlFUlRKTlZHZDZUbXBCTTFkb1kwNU5hazEzVDBSRk1rMVVaekJPYWtFelYycEJRVTFHYTNkRmQxbElDa3R2V2tsNmFqQkRRVkZaU1V0dldrbDZhakJFUVZGalJGRm5RVVZXZVM5dGRXTlRPRGwxYzBVeWJtczNla1I0UkdVdlJGcGxTR3ROT0RSalNGVXZUeklLVmxaeGVIUlFUakEwVERGYWRrVldkV1ZITkcxUGVUVlpNUzlWUVZOdVpUaGxSMXBZTlZaUWVHOHlaVlpSTkU5dVQyRlBRMEpVUlhkbloxVjBUVUUwUndwQk1WVmtSSGRGUWk5M1VVVkJkMGxJWjBSQlZFSm5UbFpJVTFWRlJFUkJTMEpuWjNKQ1owVkdRbEZqUkVGNlFXUkNaMDVXU0ZFMFJVWm5VVlZCYmpoaUNuRk5lVFZuVkdwU055dGhSbXhSUkVaRmVGUXpka280ZDBoM1dVUldVakJxUWtKbmQwWnZRVlV6T1ZCd2VqRlphMFZhWWpWeFRtcHdTMFpYYVhocE5Ga0tXa1E0ZDFoUldVUldVakJTUVZGSUwwSkdUWGRWV1ZwUVlVaFNNR05JVFRaTWVUbHVZVmhTYjJSWFNYVlpNamwwVERKa01WbFhUbnBhVjAxMldqTldhQXBaZVRoMVdqSnNNR0ZJVm1sTU0yUjJZMjEwYldKSE9UTmplVGw1V2xkNGJGbFlUbXhNYm14b1lsZDRRV050Vm0xamVUa3dXVmRrZWt3eldYZE1ha1YxQ2sxcVFUVkNaMjl5UW1kRlJVRlpUeTlOUVVWQ1FrTjBiMlJJVW5kamVtOTJURE5TZG1FeVZuVk1iVVpxWkVkc2RtSnVUWFZhTW13d1lVaFdhV1JZVG13S1kyMU9kbUp1VW14aWJsRjFXVEk1ZEUxQ1NVZERhWE5IUVZGUlFtYzNPSGRCVVVsRlFraENNV015WjNkT1oxbExTM2RaUWtKQlIwUjJla0ZDUVhkUmJ3cE9SRmw2V1dwbmQwMUVVbWxhVjFacFdXMVJNRTFVVG14Wk1sa3hUbFJhYkU1SFdtcE9WMFY0V1cxWk5VOUVXVEZOZWxKb1dXcEJWa0puYjNKQ1owVkZDa0ZaVHk5TlFVVkZRa0ZrZVZwWGVHeFpXRTVzVFVKdlIwTnBjMGRCVVZGQ1p6YzRkMEZSVlVWRVIyUXhXVmRPZWxwWFRYWmFNMVpvV1hwQlpVSm5iM0lLUW1kRlJVRlpUeTlOUVVWSFFrSkNlVnBYV25wTU0xSm9Xak5OZG1ScVFYVk5VelI1VFVSelIwTnBjMGRCVVZGQ1p6YzRkMEZSWjBWTVVYZHlZVWhTTUFwalNFMDJUSGs1TUdJeWRHeGlhVFZvV1ROU2NHSXlOWHBNYldSd1pFZG9NVmx1Vm5wYVdFcHFZakkxTUZwWE5UQk1iVTUyWWxSQ1prSm5iM0pDWjBWRkNrRlpUeTlOUVVWS1FrWkZUVlF5YURCa1NFSjZUMms0ZGxveWJEQmhTRlpwVEcxT2RtSlRPVzVrVjBacVl6Sldha3d5WkRGWlYwMTJURzFrY0dSSGFERUtXV2s1TTJJelNuSmFiWGgyWkROTmRtTnRWbk5hVjBaNldsTTFOVmxYTVhOUlNFcHNXbTVOZG1SSFJtNWplVGt5VFVNMGVFeHFTWGRQUVZsTFMzZFpRZ3BDUVVkRWRucEJRa05uVVhGRVEyY3dUbXBPYVU5RVFYZE9SMHBzV2xkS2FWcEVVWGhOTWxacVdtcFZNVTV0VlRCYWJVMHhXVlJHYVZwcWF6Uk9hbFY2Q2s1SFJtbE5RakJIUTJselIwRlJVVUpuTnpoM1FWRnpSVVIzZDA1YU1td3dZVWhXYVV4WGFIWmpNMUpzV2tSQmRrSm5iM0pDWjBWRlFWbFBMMDFCUlUwS1FrTkZUVWd5YURCa1NFSjZUMms0ZGxveWJEQmhTRlpwVEcxT2RtSlRPVzVrVjBacVl6Sldha3d5WkRGWlYwMTNUMEZaUzB0M1dVSkNRVWRFZG5wQlFncEVVVkZ4UkVObk1FNXFUbWxQUkVGM1RrZEtiRnBYU21sYVJGRjRUVEpXYWxwcVZURk9iVlV3V20xTk1WbFVSbWxhYW1zMFRtcFZlazVIUm1sTlEwRkhDa05wYzBkQlVWRkNaemM0ZDBGUk5FVkZaM2RSWTIxV2JXTjVPVEJaVjJSNlRETlpkMHhxUlhWTmFrRmFRbWR2Y2tKblJVVkJXVTh2VFVGRlVFSkJjMDBLUTFSVmQwMXFSWGxPZWtVeVRtcEJjVUpuYjNKQ1owVkZRVmxQTDAxQlJWRkNRbmROUjIxb01HUklRbnBQYVRoMldqSnNNR0ZJVm1sTWJVNTJZbE01Ymdwa1YwWnFZekpXYWsxQ2EwZERhWE5IUVZGUlFtYzNPSGRCVWtWRlEzZDNTazFVUlhoTmVtdDZUMFJyZUUxR09FZERhWE5IUVZGUlFtYzNPSGRCVWtsRkNsVlJlRkJoU0ZJd1kwaE5Oa3g1T1c1aFdGSnZaRmRKZFZreU9YUk1NbVF4V1ZkT2VscFhUWFphTTFab1dYazRkVm95YkRCaFNGWnBURE5rZG1OdGRHMEtZa2M1TTJONU9YbGFWM2hzV1ZoT2JFeHViR2hpVjNoQlkyMVdiV041T1RCWlYyUjZURE5aZDB4cVJYVk5ha0UwUW1kdmNrSm5SVVZCV1U4dlRVRkZWQXBDUTI5TlMwUlJNazB5U1RSTlJFRXdXVzFXYkZsdFNtdE9SRVY2V2xkT2JVNVVWVEphVkZKdFdYcFdhRTFYU20xUFZHY3lUbFJOTUZsWFNYZEdRVmxMQ2t0M1dVSkNRVWRFZG5wQlFrWkJVVWRFUVZKM1pGaE9iMDFHU1VkRGFYTkhRVkZSUW1jM09IZEJVbFZGVWtGNFEyRklVakJqU0UwMlRIazVibUZZVW04S1pGZEpkVmt5T1hSTU1tUXhXVmRPZWxwWFRYWmFNMVpvV1hrNWFGa3pVbkJpTWpWNlRETktNV0p1VFhaT1ZHYzBUV3BWTUU5RVNUQk5RemxvWkVoU2JBcGlXRUl3WTNrNGVFMUNXVWREYVhOSFFWRlJRbWMzT0hkQlVsbEZRMEYzUjJOSVZtbGlSMnhxVFVsSFNrSm5iM0pDWjBWRlFXUmFOVUZuVVVOQ1NITkZDbVZSUWpOQlNGVkJNMVF3ZDJGellraEZWRXBxUjFJMFkyMVhZek5CY1VwTFdISnFaVkJMTXk5b05IQjVaME00Y0Rkdk5FRkJRVWRLTHpaSmVXWm5RVUVLUWtGTlFWSnFRa1ZCYVVJNVFVZHJUMFJwZG1Nd2RFNUZNV2xwYVdGNE9XMUhVMjlHZUZocmEzaEtialJZYjBWVVkwTTNUVTlSU1dkUmJGaFZPR1p6YWdwclZYTXJSMU1yTmt0WlkxQllhRmQ2V0RCdVpIaE1SVkZFV0RGdlpUbHRNMmRXUlhkRFoxbEpTMjlhU1hwcU1FVkJkMDFFV25kQmQxcEJTWGRYZVZodUNuazJZM1UyTm00M05sRnFOVGxqZUdaSmNVRlBlV1ZsV0cxR1NrdG9iMlZKZEdaTFUzWTFaRTVHTjA4elNFUnFkR3M1T0V0aE15dHdVV2xoZWtGcVFVd0tlRnBRZFVwRldIZEZTSEZTY0RRMWJrSkpVelpDUkRkMVdFTkZaV2xhTVV4dE0xbEhhelJwYTNWakwzVjVkVGxLYW5kRlozTmxTR2h4UmtkSVZXdzRQUW90TFMwdExVVk9SQ0JEUlZKVVNVWkpRMEZVUlMwdExTMHRDZz09In19fX0=","integratedTime":1692210967,"logIndex":31541333,"logID":"c0d23d6ad406973f9559f3ba2d1ca01f84147d8ffc5b8445c224f98b9591801d"}},"Issuer":"https://token.actions.githubusercontent.com","Subject":"https://github.com/guacsec/guac/.github/workflows/release.yaml@refs/tags/v0.1.2","git_sha":"463b8004beebbd413ecf556e4fc5a1bf986534ab","githubWorkflowName":"release","githubWorkflowRef":"refs/tags/v0.1.2","githubWorkflowRepository":"guacsec/guac","githubWorkflowSha":"463b8004beebbd413ecf556e4fc5a1bf986534ab","githubWorkflowTrigger":"push"}}]
```

## Step 1: Download GUAC

1. Download the GUAC CLI `guacone` binary for your machine's OS and architecture
   from the
   [latest GUAC release](https://github.com/guacsec/guac/releases/latest). For
   example Linux x86_64 is
   [`guacone-linux-amd64'](https://github.com/guacsec/guac/releases/latest/download/guacone-linux-amd64).

1. Rename the binary to `guacone`, mark it executable if necessary, and add it
   to your shell's path.

1. Download the compose files from the
   [latest GUAC release](https://github.com/guacsec/guac/releases/latest). Look
   for an attached file `guac-compose-<release-tag>.tar.gz`. At the time of
   writing, this is:
   [`guac-compose-v0.1.1.tar.gz`](https://github.com/guacsec/guac/releases/download/v0.1.1/guac-compose-v0.1.1.tar.gz).

1. Untar the compose files and change to that directory. (the rest of the steps
   need to be done from this directory):

   ```bash
   tar zxvf guac-compose-v0.1.1.tar.gz
   cd guac-compose
   ```

1. Optional: If you want test data to use,
   [download and unzip GUAC’s test data.](https://github.com/guacsec/guac-data/archive/refs/heads/main.zip)

## Step 2: Start the GUAC server

1. In another terminal, from the `guac-compose` directory, run:

   ```bash
   docker compose up --force-recreate
   ```

2. Verify that GUAC is running:

   ```bash
   docker compose ls --filter "name=guac"
   ```

   You should see:

   ```bash
   NAME                STATUS              CONFIG FILES
   guac                running(7)          /Users/lumb/go/src/github.com/guacsec/guac/docker-compose.yml
   ```

   **If you don’t see the above,** run `docker compose down` and try starting up
   GUAC again. Because Docker Compose caches the containers used, the unclean
   state can cause issues.

### GUAC Ports

| Port Number | GUAC Component | Note                                                                                                                                                                                                            |
| ----------- | -------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| 8080        | GraphQL server | To see the GraphQL playground, visit [http://localhost:8080](http://localhost:8080)                                                                                                                             |
| 4222        | Nats           | This is where any collectors that you run will need to connect to push any docs they find. The GUAC collector command defaults to `nats://127.0.0.1:4222` for the Nats address, so this will work automatically |

## Step 3: Start Ingesting Data

You can run the `guacone collect files` ingestion command to load data into your
GUAC deployment. For example we can ingest the sample `guac-data` data. However,
you may ingest what you wish to here instead.

```bash
guacone collect files guac-data-main/docs
```

Switch back to the compose window and you will soon see that the OSV certifier
recognized the new packages and is looking up vulnerability information for
them.

## Step 4: Check that everything is ingesting and running

Run:

```bash
curl 'http://localhost:8080/query' -s -X POST -H 'content-type: application/json' \
  --data '{
    "query": "{ packages(pkgSpec: {}) { type } }"
  }' | jq
```

You should see the types of all the packages ingested

```json
{
  "data": {
    "packages": [
      {
        "type": "oci"
      },
...
```

## What is running?

Congratulations, you are now running a full GUAC deployment! Taking a look at
the `docker-compose.yaml` we can see what is actually running:

- **Nats**: Used for communication between the GUAC components. It is available
  on port `4222`.
- **Collector-Subscriber**: Helps communicate to the collectors when additional
  information is needed.
- **GraphQL Server**: Serves GUAC GraphQL queries and stores the data. As the
  in-memory backend is used, no separate backend is needed behind the server.
- **Ingestor**: Listens for things to ingest through Nats, then pushes to the
  GraphQL Server. The ingestor also runs the assembler and parser internally.
- **Image Collector**: Can pull OCI image metadata (SBOMs and attestations) from
  registries for further inspection.
- **Deps.dev Collector**: Gathers further information from
  [Deps.dev](https://deps.dev/) for supported packages.
- **OSV Certifier**: Gathers OSV vulnerability information from
  [osv.dev](https://osv.dev/) about packages.

## Next steps

The compose configuration is suitable to leave running in an environment that is
accessible to your environment for further GUAC ingestion, discovery, analysis,
and evaluation. Keep in mind that the in-memory backend is not persistent.
Explore the types of collectors available in the `collector` binary and see what
will work for your build, ingestion, and SBOM workflow. These collectors can be
run as another service that watches a location for new documents to ingest.
