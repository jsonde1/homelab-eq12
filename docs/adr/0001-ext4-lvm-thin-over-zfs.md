# ADR 0001: Selection of ext4 with LVM-Thin over ZFS for Storage Pool

- **Status:** Accepted
- **Date:** 2026-09-20
- **Deciders:** Joshua Sonde

## Context

The primary hypervisor host (`pve-node-01`) is built on a Beelink EQ12 mini PC equipped with an Intel N100 processor, a single 500GB NVMe drive, and 16GB of DDR5 RAM.

When configuring Proxmox VE, a storage filesystem had to be selected for the primary boot and guest image pool.

## Options Considered

1. **ZFS (Zettabyte File System)**
   - _Pros:_ Native copy-on-write, inline compression, bitrot protection, instant snapshotting.
   - _Cons:_ ZFS Adaptive Replacement Cache (ARC) requires significant RAM overhead (typically 1GB RAM per 1TB storage + baseline). On a 16GB RAM node, ZFS memory consumption severely constrains room for workloads. Furthermore, running ZFS on a single non-ECC NVMe drive negates key redundancy/repair features.

2. **ext4 + LVM-Thin (Selected)**
   - _Pros:_ Minimal memory/CPU overhead, highly efficient block allocation, full support for Proxmox snapshotting and thin provisioning, lightweight on single NVMe consumer hardware.
   - _Cons:_ No automatic self-healing/bitrot repair at the filesystem layer.

## Decision

We decided to format the boot drive with **ext4** and utilize **LVM-Thin** for virtual disk storage (`local-lvm`).

## Consequences

- **Positive:** Preserves ~14GB+ of host RAM strictly for container and VM allocation (Immich, Home Assistant, Docker LXC) rather than filesystem caching.
- **Mitigation:** Bitrot risk and single-disk failure risks are mitigated via scheduled, off-host backups using Proxmox Backup / PBS rather than relying on host-level ZFS mirrors.
