---
layout: page
title: Verify and validate via cosign and slsa-verifier
permalink: /verification/
parent: Getting started with GUAC
nav_order: 1
---

## Prerequisites

- [cosign](https://docs.sigstore.dev/system_config/installation/)
- [crane](https://github.com/google/go-containerregistry/blob/main/cmd/crane/README.md)
- [slsa-framework/slsa-verifier](https://github.com/slsa-framework/slsa-verifier#installation)

## Step 1: Verify GUAC image via Cosign

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

2. We can also verify the SLSA attestation on the image via:

   ```bash
   slsa-verifier verify-image ghcr.io/guacsec/guac@$GUAC_DIGEST --source-uri github.com/guacsec/guac
   ```

   you should see an output similar to this:

   ```bash
   Verified build using builder https://github.com/slsa-framework/slsa-github-generator/.github/workflows/generator_container_slsa3.yml@refs/tags/v1.8.0 at commit 463b8004beebbd413ecf556e4fc5a1bf986534ab
   PASSED: Verified SLSA provenance
   ```

## Step 2: Download GUAC binary and vertify

1. Download the GUAC CLI `guacone` binary for your machine's OS and architecture
   from the
   [latest GUAC release](https://github.com/guacsec/guac/releases/latest). For
   example Linux x86_64 is
   [`guacone-linux-amd64'](https://github.com/guacsec/guac/releases/latest/download/guacone-linux-amd64).

2. Verify the signature. We generate [SLSA 3 provenance](https://slsa.dev) using
   the OpenSSF's
   [slsa-framework/slsa-github-generator](https://github.com/slsa-framework/slsa-github-generator).
   To verify our release, install the verification tool from
   [slsa-framework/slsa-verifier#installation](https://github.com/slsa-framework/slsa-verifier#installation)
   and verify as follows:

   For example if using
   [`guacone-linux-amd64'](https://github.com/guacsec/guac/releases/latest/download/guacone-linux-amd64)
   the command would would be as follows:

   ```bash
   LATEST_VERSION=$(curl https://api.github.com/repos/guacsec/guac/releases/latest | grep tag_name | cut -d : -f2 | tr -d "v\", ")
   curl -sL https://github.com/guacsec/guac/releases/download/v$LATEST_VERSION/multiple.intoto.jsonl > multiple.intoto.jsonl
   slsa-verifier verify-artifact ./guacone-linux-amd64 --provenance-path ./multiple.intoto.jsonl --source-uri github.com/guacsec/guac
   ```

   You should see an output similar to this:

   ```bash
   Verified signature against tlog entry index 31541425 at URL: https://rekor.sigstore.dev/api/v1/log/entries/24296fb24b8ad77a020e9997ab4ec1312d9f95e211b528e9a8751f775d78c5542a365cd2bfb82871
   Verified build using builder "https://github.com/slsa-framework/slsa-github-generator/.github/workflows/generator_generic_slsa3.yml@refs/tags/v1.8.0" at commit 463b8004beebbd413ecf556e4fc5a1bf986534ab
   Verifying artifact ./guacone-linux-amd64: PASSED

   PASSED: Verified SLSA provenance
   ```
