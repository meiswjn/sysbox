# 2026-02-04: sysbox-deploy-k8s (AKS/containerd enablement)

## Goal
Improve `sysbox-pkgr/k8s` artifacts to support AKS-style containerd deployments (without CRI-O) and fix a build bug.

## Changes
- Added containerd config snippet for containerd CRI plugin key `io.containerd.grpc.v1.cri`.
- Updated `sysbox-deploy-k8s.sh` to:
  - Select the appropriate containerd config snippet based on host `/etc/containerd/config.toml`.
  - Inherit `SystemdCgroup` from the host containerd config when adding `sysbox-runc` (AKS sets it to `true`).
  - Fail fast if containerd config is missing.
  - Restart containerd after changing/restoring config (install-no-crio + cleanup without CRI-O).
- Updated `sysbox-pkgr/k8s/Dockerfile.sysbox-ce` to include the containerd config snippets under `/opt/sysbox/config/`.
- Updated `sysbox-pkgr/k8s/Makefile` to remove hardcoded `amd64` sysbox-runc overwrite and allow configurable overwrites via `OVERWRITE_SYSBOX_BINS_FROM_SOURCE`.

## Files changed
- sysbox-pkgr/k8s/config/etc_containerd_config_grpc_v1_cri (new)
- sysbox-pkgr/k8s/scripts/sysbox-deploy-k8s.sh
- sysbox-pkgr/k8s/Dockerfile.sysbox-ce
- sysbox-pkgr/k8s/Makefile

## Notes
- The repo docs still emphasize CRI-O for userns; however `install-no-crio` exists and is now functional for containerd-based clusters.
