---
layout: page
title: Learning GraphQL
permalink: /guac/learning-graphql/
parent: Developing tools on top of GUAC
nav_order: 3
---

# Learning GraphQL

GUAC exposes its software supply chain graph through a GraphQL API. If you are
new to GraphQL, these resources can help you understand queries and explore
GUAC's schema before building your own integrations.

## Start with the fundamentals

- [Learn GraphQL](https://graphql.org/learn/) introduces schemas, types,
  queries, mutations, and introspection. Start here if GraphQL is new to you.
- [GraphQL queries](https://graphql.org/learn/queries/) explains how to select
  fields and use arguments, variables, and fragments when querying an API.

## Follow a guided tutorial

- [How to GraphQL](https://www.howtographql.com/) offers free, practical
  tutorials, including GraphQL fundamentals and language-specific examples.
- [GraphQL training courses](https://graphql.org/community/training-courses/)
  curates tutorials and courses from the GraphQL community.

## Try GraphQL with GUAC

Start the GUAC GraphQL server with `guacgql --gql-debug` to enable its
playground. Use the playground's documentation explorer to discover available
queries and types, then try selecting fields from GUAC's graph.

For GUAC-specific examples, see [the GraphQL API
demo]({{ site.baseurl }}{%link guac/guac-graphql.md %}) and [GUAC GraphQL
definitions]({{ site.baseurl }}{%link guac/graphql.md %}). The resources above
teach general GraphQL concepts; the GUAC pages describe GUAC's own API.
