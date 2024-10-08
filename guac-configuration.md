---
layout: page
title: GUAC Configuration Guide
permalink: /guac-configuration/
parent: Getting started with GUAC
nav_order: 7
---

# GUAC Configuration Guide

This document provides an overview of the `guac.yaml` configuration file used in GUAC deployments. It describes each configuration option, its default value, and scenarios where you might want to change it.

## Database Configuration

### ArangoDB
- **arango-user**: `root`
  - **Description**: The username for connecting to the ArangoDB instance.
  - **When to Change**: Change this if you have a different user set up for security reasons or if using a managed ArangoDB service.
  
- **arango-pass**: `test123`
  - **Description**: The password for the ArangoDB user.
  - **When to Change**: Always change the default password to a secure one in production environments.

- **arango-addr**: `http://localhost:8529`
  - **Description**: The address of the ArangoDB server.
  - **When to Change**: Modify this if your ArangoDB server is hosted on a different machine or if hosted on the cloud.

### Neo4j
- **neo4j-user**: `neo4j`
  - **Description**: The username for connecting to the Neo4j database.
  - **When to Change**: Use a different user for enhanced security or if your database setup requires it.

- **neo4j-pass**: `s3cr3t`
  - **Description**: The password for the Neo4j user.
  - **When to Change**: Change to a secure password for production use.

- **neo4j-addr**: `neo4j://localhost:7687`
  - **Description**: The address of the Neo4j server.
  - **When to Change**: Update this if your Neo4j instance is not running locally or is hosted in the cloud.

- **neo4j-realm**: `neo4j`
  - **Description**: The realm for the Neo4j database, typically used for multi-tenancy.
  - **When to Change**: Adjust if your setup uses a different realm for authentication.

### Neptune
- **neptune-user**: `username`
  - **Description**: The username for connecting to the Neptune database.
  - **When to Change**: Change this to match your Neptune database credentials.

- **neptune-endpoint**: `localhost`
  - **Description**: The endpoint for the Neptune database.
  - **When to Change**: Update this to the actual endpoint if using AWS Neptune.

- **neptune-port**: `8182`
  - **Description**: The port for the Neptune database.
  - **When to Change**: Change if your Neptune instance uses a non-default port.

- **neptune-region**: `us-east-1`
  - **Description**: The AWS region where the Neptune database is hosted.
  - **When to Change**: Modify to match the region of your AWS Neptune instance.

- **neptune-realm**: `neptune`
  - **Description**: The realm for the Neptune database.
  - **When to Change**: Adjust if your setup uses a different realm.

## Pub/Sub Configuration

- **pubsub-addr**: `nats://localhost:4222`
  - **Description**: The address of the NATS server for pub/sub messaging.
  - **When to Change**: Update if your NATS server is hosted elsewhere or uses a different port.

- **publish-to-queue**: `true`
  - **Description**: Whether to publish messages to the queue.
  - **When to Change**: Set to `false` if you do not want to use queue-based messaging.

## Blob Store Configuration

- **blob-addr**: `file:///tmp/blobstore?no_tmp_dir=true`
  - **Description**: The address of the blob store.
  - **When to Change**: Change to use a different storage backend, such as AWS S3 or Google Cloud Storage (examples).

## Certifier Configuration

- **interval**: `20m`
  - **Description**: The interval at which the certifier runs.
  - **When to Change**: Adjust based on how frequently you need certification checks.

- **last-scan**: `4`
  - **Description**: The number of hours since the last scan was run. A value of `0` means the scan will run on all packages/sources.
  - **When to Change**: Set to `0` for a full scan or adjust based on your scanning frequency needs.

- **certifier-batch-size**: `60000`
  - **Description**: The batch size for the package pagination query.
  - **When to Change**: Modify to optimize performance based on your system's capabilities.

- **certifier-latency**: `""`
  - **Description**: Artificial latency to throttle the certifier.
  - **When to Change**: Use to introduce delays if needed to manage load.

## Deps.dev Configuration

- **deps-dev-latency**: `""`
  - **Description**: Artificial latency to throttle deps.dev.
  - **When to Change**: Adjust if you need to manage load on deps.dev queries.

## Ingestion Configuration

- **add-vuln-on-ingest**: `false`
  - **Description**: Whether to query vulnerabilities during ingestion.
  - **When to Change**: Set to `true` if you want to automatically check for vulnerabilities during data ingestion.

- **add-license-on-ingest**: `false`
  - **Description**: Whether to query licenses during ingestion.
  - **When to Change**: Set to `true` if you want to automatically check for licenses during data ingestion.

## CSub Configuration

- **csub-addr**: `localhost:2782`
  - **Description**: The address for the CSub service.
  - **When to Change**: Update if your CSub service is hosted on a different address.

- **csub-listen-port**: `2782`
  - **Description**: The port on which the CSub service listens.
  - **When to Change**: Change if your CSub service uses a different port.

## GraphQL Configuration

- **gql-backend**: `keyvalue`
  - **Description**: The backend used for the GraphQL server.
  - **When to Change**: Modify if using a different backend for GraphQL.

- **gql-listen-port**: `8080`
  - **Description**: The port on which the GraphQL server listens.
  - **When to Change**: Change if your GraphQL server uses a different port.

- **gql-debug**: `true`
  - **Description**: Whether to enable debug mode for the GraphQL server.
  - **When to Change**: Set to `false` in production environments for security.

- **gql-addr**: `http://localhost:8080/query`
  - **Description**: The address of the GraphQL server.
  - **When to Change**: Update if your GraphQL server is hosted elsewhere.

## REST API Configuration

- **rest-api-server-port**: `8081`
  - **Description**: The port on which the API server listens.
  - **When to Change**: Change if your API server uses a different port.

## Collector Configuration

- **service-poll**: `true`
  - **Description**: Whether the collector should poll services.
  - **When to Change**: Set to `false` if polling is not required.

- **use-csub**: `true`
  - **Description**: Whether to use the CSub service.
  - **When to Change**: Set to `false` if not using CSub.

## Logging Configuration

- **log-level**: `Info`
  - **Description**: The logging level for the application.
  - **When to Change**: Adjust to `Debug` for more detailed logs during development or troubleshooting.
