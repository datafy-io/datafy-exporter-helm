# Chart changes

## 0.1.1

- Remove the separate pipes-directory hostPath; exporter 0.0.6 reopens current
  geometry state through the host-root mount after an agent restart.
- Add `stateDir` for custom agent state bases; Bottlerocket is auto-detected.
- Allow six seconds for metrics probes to cover bounded disk and FIFO waits.

- Default to exporter 0.0.6, which fixes host mount labels, volume IDs after PVC
  moves and undatafy, and stalled scrapes during agent-core outages.
- DT-12919: Set `DATAFY_EXPORTER_HOST_ROOT=/host` for corrected host mount labels.
- DT-12920: Propagate new and moved host mounts into the running exporter pod.

Publish exporter image 0.0.6 before releasing this chart. Queries matching old
`/host...` mount labels need to use the corrected host paths after upgrading.
