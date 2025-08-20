---
layout: page
title: Set up the GUAC Visualizer
permalink: /guac/guac-visualizer/
redirect_from: /guac-visualizer/
parent: Getting started with GUAC
nav_order: 5
---

# Set up the GUAC Visualizer

The GUAC Visualizer is an experimental utility that can be used to interact with
GUAC services. It acts as a way to visualize the software supply chain graph, as
well as a means to explore the supply chain and prototype policies.

{: .no_toc }

<details open markdown="block">
  <summary>
    Table of contents
  </summary>
  {: .text-delta }
- TOC
{:toc}
</details>

## Requirements

- [git](https://git-scm.com/downloads)
- [yarn](https://classic.yarnpkg.com/lang/en/docs/install/#mac-stable)
- [node](https://docs.npmjs.com/downloading-and-installing-node-js-and-npm)
  version >= 18
- [GUAC services up and running]({{
  site.baseurl }}{%link guac/setup.md %})

## Installation

### Step 1. Getting started

Get the guac-visualizer
[source code here](https://github.com/guacsec/guac-visualizer/releases/latest).

`cd` into it:

```bash
cd guac-visualizer
```

### Step 2. Install dependencies

```bash
yarn install
```

### Step 3. Run the visualizer locally:

```bash
yarn dev
```

You can then go to [localhost:3000](http://localhost:3000) in your browser to
start using the visualizer.

```
...

$ next dev
ready - started server on 0.0.0.0:3000, url: http://localhost:3000
info  - Using webpack 5. Reason: Enabled by default https://nextjs.org/docs/messages/webpack5

...
```

<hr />

**Using the GUAC visualizer will look something like this:**

![image](https://github.com/guacsec/guac-visualizer/assets/68356865/420c523e-9774-4a4f-82c1-b7e1d29ba9ac)

<hr />

## Configuration

When running the development server , or as a container app, the visualizer app
server needs some configuration. The default configuration is stored in the
`.env` file.

The most important configuration setting is `NEXT_PUBLIC_GUACGQL_SERVER_URL`,
which points to the running GUAC GraphQL server. The default configuration file
uses `http://localhost:8080`.

You have multiple options to change the configuration to your local needs:

- provide environment variable (typical for execution environment, like
  kubernetes)
- create file `.env.local`, which then contains local configurations (typical
  during development)

If these two options are not fitting for you, there are even more options for
changing the configuration, please read the next.js documentation
[chapter on environment variables](https://nextjs.org/docs/app/building-your-application/configuring/environment-variables).

## Troubleshooting

If you get an error when guac-visualizer tries to render, you may need to update
the generated graphql code. To do this:

1. Clone the [GUAC repo](https://github.com/guacsec/guac)
2. Ensure the paths in the `codegen.ts` file in this repository are correct. If
   you cloned GUAC into a directory next to this repo's directory, you will not
   need to change the paths.
3. Run `yarn graphql-codegen` to update the GraphQL code.
