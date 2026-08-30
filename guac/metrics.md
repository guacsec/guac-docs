---
layout: page
title: Metrics and Observability
permalink: /guac/metrics/
redirect_from: /metrics/
parent: How GUAC works
nav_order: 6
---

# Metrics and Observability in GUAC

{: .no_toc }

<details open markdown="block">
  <summary>
    Table of contents
  </summary>
  {: .text-delta }
- TOC
{:toc}
</details>

GUAC provides robust observability features to help operators monitor service
health, track ingestion throughput, measure request latency, and debug supply
chain graph operations. Observability in GUAC is supported through two
complementary systems:

1. **OpenTelemetry (OTel)**: For distributed tracing and metrics instrumentation
   across integrated libraries and network clients.
2. **Prometheus**: For exporting application, HTTP server, and custom GUAC
   metrics on a `/metrics` scrape endpoint.

---

## OpenTelemetry (OTel) Metrics and Tracing

GUAC uses OpenTelemetry instrumented libraries for key system components:

- **GraphQL Server (`guacgql`)**: HTTP request tracing and server metrics.
- **Database Backend**: SQL driver instrumentation underneath the Ent/PostgreSQL
  backend.
- **External HTTP Clients**: Instrumented HTTP clients for OSV, ClearlyDefined,
  GitHub, and End-of-Life (EoL) services.
- **gRPC Clients**: Instrumented gRPC client for Deps.dev.

### Enabling OpenTelemetry

Any GUAC CLI or service that runs one of the above components supports the
`--enable-otel` CLI flag (or `GUAC_ENABLE_OTEL=true` environment variable) to
bootstrap the default OpenTelemetry trace and metric providers.

When enabled, GUAC configures the OTel SDK to export metrics and traces to an
OpenTelemetry collector over gRPC.

### Configuration

OpenTelemetry in GUAC is configured using standard OpenTelemetry environment
variables:

| Environment Variable          | Description                                                         | Default / Example              |
| :---------------------------- | :------------------------------------------------------------------ | :----------------------------- |
| `OTEL_EXPORTER_OTLP_ENDPOINT` | Address of the OTel collector endpoint to connect to                | `http://localhost:4317`        |
| `OTEL_EXPORTER_OTLP_INSECURE` | If `true`, disables TLS for connecting to local/insecure collectors | `true` (for local dev)         |
| `OTEL_SERVICE_NAME`           | Service name identifier attached to exported metrics and traces     | `guacgql`, `guaccollect`, etc. |

For additional details on OTel configuration and exporters, see:

- [OpenTelemetry Go OTLP Metric gRPC Exporter](https://pkg.go.dev/go.opentelemetry.io/otel/exporters/otlp/otlpmetric/otlpmetricgrpc)
- [OpenTelemetry SDK Environment Variable Specification](https://opentelemetry.io/docs/languages/sdk-configuration/general/#otel_traces_sampler)

{: .note }

> GUAC is not currently set up to define and publish custom application-level
> metrics directly to OpenTelemetry. Custom application metrics are managed
> through GUAC's `MetricCollector` interface with the Prometheus backend.

---

## Prometheus Metrics

Prometheus metrics are available across several GUAC CLI tools (including
`guacgql`, `guaccollect`, `guacingest`, `guaccsub`, and `guacone`) using the
`--enable-prometheus` option.

### Enabling Prometheus

- **`guacgql`**: When `--enable-prometheus` is passed, Prometheus metrics are
  registered directly on the GraphQL HTTP server and served on `/metrics` (e.g.,
  `http://localhost:8080/metrics`).
- **Collectors and Workers (`guaccollect`, `guacingest`, etc.)**: Passing
  `--enable-prometheus` starts an HTTP server listening on `--prometheus-port`
  (default: `9091`) serving the `/metrics` endpoint.

### Command-Line Flags

| Flag                  | Description                                      | Default |
| :-------------------- | :----------------------------------------------- | :------ |
| `--enable-prometheus` | Enables the Prometheus metrics HTTP handler      | `false` |
| `--prometheus-port`   | Port on which the Prometheus HTTP server listens | `9091`  |

### Scraping Metrics

To scrape and inspect metrics from Prometheus, run:

```bash
curl http://localhost:8080/metrics
```

Or for collectors and workers using the dedicated Prometheus port:

```bash
curl http://localhost:9091/metrics
```

### Example Scrape Output

The output is formatted in the standard Prometheus text format, containing
standard Go runtime statistics alongside custom GUAC operational metrics:

```text
# HELP go_memstats_alloc_bytes Number of bytes allocated and still in use.
# TYPE go_memstats_alloc_bytes gauge
go_memstats_alloc_bytes 6.25188e+06
# HELP go_memstats_stack_sys_bytes Number of bytes obtained from system for stack allocator.
# TYPE go_memstats_stack_sys_bytes gauge
go_memstats_stack_sys_bytes 884736
# HELP go_threads Number of OS threads created.
# TYPE go_threads gauge
go_threads 10
# HELP guac_guacgql_http_server_request_duration_seconds Histogram of response time for handler in seconds
# TYPE guac_guacgql_http_server_request_duration_seconds histogram
guac_guacgql_http_server_request_duration_seconds_bucket{method="POST",route="",status_code="200",le="0.005"} 12
guac_guacgql_http_server_request_duration_seconds_bucket{method="POST",route="",status_code="200",le="0.01"} 18
guac_guacgql_http_server_request_duration_seconds_bucket{method="POST",route="",status_code="200",le="0.025"} 25
guac_guacgql_http_server_request_duration_seconds_bucket{method="POST",route="",status_code="200",le="+Inf"} 25
guac_guacgql_http_server_request_duration_seconds_sum{method="POST",route="",status_code="200"} 0.142
guac_guacgql_http_server_request_duration_seconds_count{method="POST",route="",status_code="200"} 25
# HELP guac_http_deps_dev_version_errors Counter for http_deps_dev_version_errors
# TYPE guac_http_deps_dev_version_errors counter
guac_http_deps_dev_version_errors{name="antlr",namespace="github.com/antlr/antlr4/runtime/go",pkgtype="golang"} 2
guac_http_deps_dev_version_errors{name="api",namespace="github.com/hashicorp/vault",pkgtype="golang"} 1
guac_http_deps_dev_version_errors{name="consul",namespace="github.com/hashicorp",pkgtype="golang"} 1
guac_http_deps_dev_version_errors{name="name",namespace="namespace",pkgtype="pkgtype"} 0
guac_http_deps_dev_version_errors{name="readline",namespace="github.com/chzyer",pkgtype="golang"} 1
guac_http_deps_dev_version_errors{name="sdk",namespace="github.com/hashicorp/vault",pkgtype="golang"} 1
```

---

## Developer Guide: Adding Metrics to GUAC Packages

GUAC provides a modular `pkg/metrics` package with clean Go interfaces. This
makes it easy to add custom instrumentation to new packages and write unit tests
using mock collectors.

### Core Interfaces

Defined in `pkg/metrics/metrics.go`:

- **`MetricCollector`**: Main interface providing methods for registering and
  recording metrics (`RegisterHistogram`, `RegisterGauge`, `RegisterCounter`,
  `ObserveHistogram`, `SetGauge`, `AddCounter`, `MeasureFunctionExecutionTime`,
  `MetricsHandler`, `MeasureGraphQLResponseDuration`).
- **`Counter`**: Interface supporting incrementing (`Inc()`) and adding
  (`Add(float64)`).
- **`Observable`**: Interface for observing values in histograms
  (`Observe(float64)`).

### Instrumenting a Package

To instrument a new GUAC package or collector:

1. **Import the metrics package**:

   ```go
   import "github.com/guacsec/guac/pkg/metrics"
   ```

2. **Obtain or initialize the `MetricCollector`**:

   ```go
   // From context or by instantiating a new Prometheus collector
   m := metrics.FromContext(ctx, "my_collector")
   ```

3. **Register metrics**:

   ```go
   // Register a counter
   counter, err := m.RegisterCounter(ctx, "ingestion_errors_total", "doc_type")
   if err != nil {
       return fmt.Errorf("failed to register counter: %w", err)
   }

   // Register a gauge
   gauge, err := m.RegisterGauge(ctx, "active_workers", "worker_pool")
   if err != nil {
       return fmt.Errorf("failed to register gauge: %w", err)
   }

   // Register a histogram
   histogram, err := m.RegisterHistogram(ctx, "processing_duration_seconds", "parser_type")
   if err != nil {
       return fmt.Errorf("failed to register histogram: %w", err)
   }
   ```

4. **Update metrics and measure performance**:

   ```go
   // Increment or add to a counter
   m.AddCounter(ctx, "ingestion_errors_total", 1, "cyclonedx")

   // Set gauge value
   m.SetGauge(ctx, "active_workers", 4, "default")

   // Observe histogram
   m.ObserveHistogram(ctx, "processing_duration_seconds", 0.045, "spdx")

   // Measure function execution duration
   defer m.MeasureFunctionExecutionTime(ctx, "parseDocument")()
   ```

5. **Expose the HTTP handler**:

   ```go
   http.Handle("/metrics", m.MetricsHandler())
   ```

---

## Next Steps

- Review the [GUAC Configuration
  Guide]({{ site.baseurl }}{% link guac/guac-configuration.md %}) to configure
  metrics and service ports in `guac.yaml`.
- Learn more about component architecture in [How GUAC components work
  together]({{ site.baseurl }}{% link guac/guac-components.md %}).
- Explore the [GUAC GitHub repository](https://github.com/guacsec/guac) to
  contribute new metrics or collectors.
