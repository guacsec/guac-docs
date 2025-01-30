---
layout: page
title: Ingest Data
permalink: /setup-ingest-data/
parent: Start a demo GUAC with Docker Compose
nav_order: 2
---

# Ingest data

You can run the `guacone collect files` ingestion command to load data into your
GUAC deployment. For example we can ingest the sample `guac-data` data. However,
you may ingest what you wish to here instead.

```bash
guacone collect --add-vuln-on-ingest --add-eol-on-ingest --add-license-on-ingest files guac-data-main/docs
```

{: .note }

The `--add*` flags above will cause GUAC to query external services for
additional data while ingesting the files. This will slow down the ingestion
time. Alternatively, you can leave off those flags and run each certifier
individually (e.g. `guacone certifier osv` to get vulnerability data) as
desired.

Switch back to the compose window and you will soon see that the OSV certifier
recognized the new packages and is looking up vulnerability information for
them.

## Check that everything is ingesting and running

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

- **Collector-Subscriber**: Helps communicate to the collectors when additional
  information is needed.
- **GraphQL Server**: Serves GUAC GraphQL queries and stores the data. As the
  in-memory backend is used, no separate backend is needed behind the server.
- **Deps.dev Collector**: Gathers further information from
  [Deps.dev](https://deps.dev/) for supported packages.
- **OSV Certifier**: Gathers OSV vulnerability information from
  [osv.dev](https://osv.dev/) about packages.

## Next steps

Now it's time to start exploring the GUAC demos. Start by [expanding your view
of the software supply chain]({{
site.baseurl }}{%link expanding-your-view.md %}).

This compose configuration is suitable to leave running in an environment that
is accessible to your environment for the GUAC demos and further GUAC ingestion,
discovery, analysis, and evaluation. Keep in mind that the in-memory backend is
not persistent. Explore the types of collectors available under the
`guacone collect` command and see what will work for your build, ingestion, and
SBOM workflow. These collectors can be run as another service that watches a
location for new documents to ingest. If you’re curious about the various GUAC
components and what they do, see [How GUAC components work together]({{
site.baseurl }}{%link guac-components.md %}).
