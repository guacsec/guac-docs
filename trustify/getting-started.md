---
title: Getting Started
layout: page
permalink: /trustify/getting-started
parent: Trustify Docs
nav_order: 1
---

# Getting Started

This guide will walk you through the process of setting up Trustify on your
local machine and analyzing your first Software Bill of Materials (SBOM). We'll
be using the pre-built binaries, which is the quickest and easiest way to get
started.

## Prerequisites

You don't need any special tools to run Trustify. You'll just need a way to
download a file and run it from your terminal. We'll use `curl` in this guide
for downloading and interacting with the API, but you can use any similar tool.

## 1. Download Trustify

The easiest way to get Trustify is to download the latest pre-built binary for
your operating system from the
[Trustify Releases](https://github.com/guacsec/trustify/releases) page.

Look for the `trustd-pm` binaries in the "Assets" section of the latest release.
Download the one that matches your system (e.g.,
`trustd-pm-...-x86_64-unknown-linux-gnu` for Linux).

Once downloaded, you may need to make the file executable. In your terminal,
run:

```shell
chmod +x /path/to/your/downloaded/trustd-pm
```

## 2. Run Trustify

Now, you can start Trustify in "Personal Machine" (PM) mode. This is a
lightweight mode that's perfect for local use. It will create a local database
in a `.trustify/` directory in your current folder.

To start Trustify, simply run the binary from your terminal:

```shell
./path/to/your/downloaded/trustd-pm
```

You should see some log output, and the server will be running in the
background.

## 3. Access the Trustify UI and API

With Trustify running, you can now access its features through your web browser
or via the REST API.

- **To use the GUI**, navigate to:
  [http://localhost:8080](http://localhost:8080)
- **To use the REST API**, navigate to:
  [http://localhost:8080/openapi/](http://localhost:8080/openapi/)

Take a moment to explore the web UI. You'll see that it's currently empty
because we haven't ingested any data yet.

## 4. Ingest Your First SBOM

Trustify is most useful when it has data to analyze. Let's upload your first
SBOM. You can use any CycloneDX or SPDX JSON file you have. If you don't have
one handy, you can use an example from the Trustify repository.

To upload an SBOM from a local file, run the following command in your terminal:

```shell
curl -X POST --data-binary @/path/to/your/sbom.json -H "Content-Type: application/json" http://localhost:8080/api/v2/sbom
```

If the upload is successful, you'll see a confirmation message. Now, if you
refresh the Trustify UI in your browser, you should see the SBOM you just
uploaded.

## 5. Ingest a Dataset

While uploading individual SBOMs is useful, you can also ingest entire datasets
of SBOMs and security advisories at once. The Trustify repository includes
several example datasets. Let's download and ingest `ds3`, which contains a
collection of Red Hat advisories and related SBOMs.

1.  **Download the dataset**:

    ```shell
    curl -L -o ds3.zip https://github.com/guacsec/trustify/raw/main/etc/datasets/ds3.zip
    ```

2.  **Upload the dataset to Trustify**:
    ```shell
    curl -X POST --data-binary @ds3.zip -H "Content-Type: application/zip" http://localhost:8080/api/v2/dataset
    ```

After the upload is complete, refresh the Trustify UI. You will now see a much
richer set of data to explore, including advisories and multiple SBOMs.

## 6. Next Steps

Congratulations! You've successfully set up Trustify and ingested your first
SBOM. From here, you can start to explore the power of Trustify:

- **Upload more SBOMs and security advisories** to build a comprehensive picture
  of your software.
- **Explore the relationships** between your software components and known
  vulnerabilities in the UI.
- **Use the REST API** to automate your software supply chain security
  workflows.

For more advanced topics, such as configuring authentication or setting up data
importers, please refer to the rest of our documentation.

---

### Alternative for Developers: Building from Source

If you are a developer and want to build Trustify from source, you'll need a
recent version of Rust and `cargo`.

1.  **Clone the repository**:

    ```shell
    git clone https://github.com/guacsec/trustify.git
    cd trustify
    ```

2.  **Run Trustify**:
    ```shell
    AUTH_DISABLED=true cargo run --bin trustd
    ```

This will build and run Trustify in the same "PM mode" as the binary.
