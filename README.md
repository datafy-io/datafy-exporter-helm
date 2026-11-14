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
| `hostPort.enabled` | `true` | Expose metrics port on node IP (required for remote Prometheus) |
| `resources.limits.cpu` | `100m` | CPU limit |
| `resources.limits.memory` | `64Mi` | Memory limit |
| `resources.requests.cpu` | `50m` | CPU request |
| `resources.requests.memory` | `32Mi` | Memory request |

Override any value with `--set` or a custom `values.yaml`:

```sh
helm install datafy-exporter . -n datafy --set port=9100
```

## Host mount labels (DT-12919)

The chart sets `DATAFY_EXPORTER_HOST_ROOT=/host` to match its host-root volume
mount. With an exporter image containing DT-12919, disk metrics use the node's
mount paths: `/host` becomes `/`, and `/host/boot/efi` becomes `/boot/efi`.
Native installations leave this variable unset and retain their existing paths.

Select the image containing the fix with `image.tag` until the chart's default
`appVersion` is updated for that release. Exporter 0.0.5 predates this setting.
Existing Prometheus series with `/host...` labels stop receiving samples; update
queries that explicitly match those paths to the corrected host paths.

## Prometheus Scraping

The exporter binds to each node's IP via `hostPort`, so remote Prometheus instances can scrape it directly using node discovery:

```yaml
scrape_configs:
  - job_name: datafy-exporter
    kubernetes_sd_configs:
      - role: node
    relabel_configs:
      - source_labels: [__address__]
        regex: '(.*):.*'
        replacement: '$1:9908'
        target_label: __address__
```

Set `hostPort.enabled=false` only if Prometheus runs inside the cluster and scrapes pod IPs directly.

## Reference

- [Datafy Metrics Exporter docs](https://docs.datafy.io/resources/datafy-metrics-exporter)
