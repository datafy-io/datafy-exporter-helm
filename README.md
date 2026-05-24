# datafy-exporter-helm

Helm chart for the [Datafy Metrics Exporter](https://docs.datafy.io/resources/datafy-metrics-exporter) — a DaemonSet that exposes Datafy agent health and autoscaling volume metrics to Prometheus.

## Overview

The exporter runs on every node and provides two endpoints:

| Endpoint | Description |
|---|---|
| `/metrics` | Volume size and utilization stats for autoscaling volumes |
| `/health` | Datafy agent liveness status |

Metrics include disk capacity, available bytes, utilization percentage, and volume identification labels (`/dev/nvme*`, `vol-*`, mount points).

> **Note:** For out-of-space monitoring of autoscaling volumes, use filesystem-reported utilization rather than exporter metrics.

## Prerequisites

- Kubernetes 1.19+
- Helm 3.x
- Datafy agent running on the target nodes

## Installation

```sh
helm install datafy-exporter . -n datafy --create-namespace
```

Upgrade:

```sh
helm upgrade datafy-exporter . -n datafy
```

## Configuration

| Parameter | Default | Description |
|---|---|---|
| `image.repository` | `public.ecr.aws/datafy-io/datafy-exporter` | Image repository |
| `image.tag` | `""` (uses `appVersion`) | Image tag |
| `image.pullPolicy` | `IfNotPresent` | Pull policy |
| `port` | `9908` | Metrics port |
| `hostPort.enabled` | `false` | Expose port on node network interface |
| `resources.limits.cpu` | `100m` | CPU limit |
| `resources.limits.memory` | `64Mi` | Memory limit |
| `resources.requests.cpu` | `50m` | CPU request |
| `resources.requests.memory` | `32Mi` | Memory request |

Override any value with `--set` or a custom `values.yaml`:

```sh
helm install datafy-exporter . -n datafy --set port=9100
```

## Prometheus Scraping

The exporter is compatible with standard Prometheus scraping. Example scrape config:

```yaml
scrape_configs:
  - job_name: datafy-exporter
    kubernetes_sd_configs:
      - role: pod
    relabel_configs:
      - source_labels: [__meta_kubernetes_pod_label_app]
        action: keep
        regex: datafy-exporter
      - source_labels: [__meta_kubernetes_pod_ip]
        target_label: __address__
        replacement: $1:9908
```

Enable `hostPort.enabled=true` only if your Prometheus scrapes node IPs directly rather than pod IPs.

## Reference

- [Datafy Metrics Exporter docs](https://docs.datafy.io/resources/datafy-metrics-exporter)
