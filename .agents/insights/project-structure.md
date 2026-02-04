# Sysbox Project Structure

**Created:** 2026-02-04  
**Last Updated:** 2026-02-04

## Validation Notes (2026-02-04)

- Kernel requirements in the K8s deploy script are more specific than the earlier summary: Ubuntu requires kernel >= 5.3, and other distros require >= 5.5 (see `sysbox-pkgr/k8s/scripts/sysbox-deploy-k8s.sh`). The previous "5.4+ (5.12+ recommended)" note may be outdated or context-dependent.
- Go toolchain requirements vary by component in this workspace:
  - `sysbox-runc/go.mod` and `sysbox-mgr/go.mod` specify `go 1.24.0`.
  - Most other modules specify `go 1.21` or `go 1.22`.
  - Some syscall test modules specify older `go` versions.
  The earlier "Go 1.21+" summary is incomplete.

## Overview

Sysbox is an open-source container runtime that enhances containers with:
- Linux user-namespace isolation (root in container ≠ root on host)
- Virtualized procfs & sysfs
- Ability to run system workloads (systemd, Docker, K8s) inside containers

Originally developed by Nestybox, acquired by Docker in 2022.

## Repository Structure

```
sysbox/                     # Main repository (umbrella)
├── sysbox-runc/           # Fork of OCI runc with sysbox enhancements
├── sysbox-mgr/            # System manager daemon
├── sysbox-fs/             # Virtual filesystem (FUSE-based)
├── sysbox-libs/           # Shared Go libraries
├── sysbox-ipc/            # gRPC communication between components
├── sysbox-pkgr/           # Packaging (deb, rpm, K8s)
├── sysbox-dockerfiles/    # Dockerfiles for system containers
├── sysbox-k8s-manifests/  # K8s deployment manifests
├── docs/                  # Documentation
├── tests/                 # Integration tests (bats)
└── scr/                   # Helper scripts
```

## Component Details

### sysbox-runc

**Purpose:** OCI-compliant container runtime with user namespace enhancements

**Key Directories:**
- `libcontainer/` - Core container management (fork of runc's libcontainer)
- `libsysbox/` - Sysbox-specific extensions
  - `syscont/spec.go` - OCI spec configuration for system containers
  - `sysbox/mgr.go` - Communication with sysbox-mgr
  - `sysbox/fs.go` - Communication with sysbox-fs
- `libcontainer/internal/userns/` - User namespace handling

**Key Files:**
- `libsysbox/syscont/spec.go` - Main entry for spec validation/configuration
- `libcontainer/specconv/spec_linux.go` - OCI to libcontainer config conversion
- `libcontainer/process_linux.go` - Process creation and namespace setup

### sysbox-mgr

**Purpose:** Daemon that manages system-wide resources for Sysbox containers

**Responsibilities:**
- UID/GID subid allocation (from /etc/subuid, /etc/subgid)
- Rootfs cloning for copy-on-write
- Volume management
- Container registration/tracking

**Key Files:**
- `mgr.go` - Main manager implementation
- `subidAlloc/` - Subid allocation logic
- `volMgr/` - Volume management
- `rootfsCloner/` - Rootfs cloning

### sysbox-fs

**Purpose:** FUSE-based virtual filesystem that virtualizes /proc and /sys

**Key Directories:**
- `handler/` - Handlers for different procfs/sysfs paths
- `fuse/` - FUSE filesystem implementation
- `process/` - Process information gathering
- `state/` - Container state management

**Key Files:**
- `process/process.go` - Process info including uid_map/gid_map reading
- `handler/implementations/` - Individual handlers for /proc entries

### sysbox-libs

**Purpose:** Shared libraries used by all sysbox components

**Key Libraries:**
- `idShiftUtils/` - UID/GID shifting utilities
- `linuxUtils/` - Linux-specific utilities
- `dockerUtils/` - Docker interaction utilities
- `capability/` - Linux capabilities handling
- `shiftfs/` - Shiftfs management

### sysbox-ipc

**Purpose:** gRPC definitions and client libraries for inter-component communication

**Structure:**
- `sysboxMgrGrpc/` - gRPC client/server for sysbox-mgr
- `sysboxFsGrpc/` - gRPC client/server for sysbox-fs
- `sysboxMgrLib/` - Data structures for sysbox-mgr IPC

### sysbox-pkgr

**Purpose:** Packaging for different deployment targets

**Structure:**
- `deb/` - Debian package creation
- `rpm/` - RPM package creation
- `k8s/` - Kubernetes DaemonSet deployment
  - `Makefile` - Build sysbox-deploy-k8s image
  - `scripts/sysbox-deploy-k8s.sh` - Installation script
  - `Dockerfile.sysbox-ce` - Deploy container image
- `systemd/` - Systemd service files

## Build System

### Main Makefile (root)
```bash
make sysbox           # Build all components
make sysbox-runc      # Build just sysbox-runc
make test             # Run tests
```

### K8s Deployment (sysbox-pkgr/k8s)
```bash
sudo make all         # Build sysbox-deploy-k8s image
```

This:
1. Builds CRI-O binaries for supported versions
2. Downloads sysbox binaries from releases
3. Optionally overwrites sysbox-runc with locally built version
4. Creates Docker image `ghcr.io/nestybox/sysbox-deploy-k8s:v<version>`

## Dependencies

### Runtime Dependencies
- Linux kernel version depends on distro / deployment method. For K8s deployments, the deploy script enforces Ubuntu >= 5.3 and non-Ubuntu >= 5.5.
- CRI-O (for K8s deployments)
- FUSE (for sysbox-fs)

### Build Dependencies
- Go toolchain varies by component (see validation notes above).
- Docker (for building deployment images)
- Make

## Configuration

### sysbox-mgr Config
- `/etc/sysbox/sysbox-mgr.conf` - Manager configuration
- `/etc/subuid`, `/etc/subgid` - Subid ranges

### sysbox-fs Config
- `/var/lib/sysboxfs/` - Virtual filesystem mount points

### K8s Deployment
- Environment variables in DaemonSet
- `/etc/crio/crio.conf` - CRI-O runtime configuration

## Testing

Tests are primarily in `tests/` using BATS (Bash Automated Testing System):
- `tests/docker/` - Docker integration tests
- `tests/pods/` - K8s pod tests
- `tests/sysmgr/` - sysbox-mgr tests
- `tests/sysfs/` - sysbox-fs tests

## Version Information

- Version stored in `VERSION` file
- K8s compatibility in `sysbox-pkgr/k8s/scripts/sysbox-deploy-k8s.sh` function `is_supported_k8s_version()`
- Currently supports: K8s 1.29, 1.30, 1.31, 1.32, 1.33 (as of master branch)
