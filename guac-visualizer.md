---
layout: page
title: Set up the GUAC Visualizer
permalink: /guac-visualizer/
parent: Getting started with GUAC
nav_order: 2
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
- [GUAC services up and running]({{
  site.baseurl }}{%link setup.md %})

## Step 1. Getting started

Get the
[source code for guac-visualizer here](https://github.com/guacsec/guac-visualizer/releases/tag/latest).

`cd` into it:

```bash
cd guac-visualizer
```

## Step 2. Install dependencies

```bash
yarn install
```

## Step 3. Run the visualizer locally:

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
