# Enterprise Edge Homelab: Beelink EQ12 (Intel N100)

A power-efficient, headless hybrid virtualization and container platform designed for local data sovereignty, smart home orchestration, and hands-on enterprise networking (CCNA) experimentation.

---

## 1. Project Overview

This repository documents the architecture, infrastructure-as-code configurations, and operational workflows for a low-power home server running on Proxmox VE.

The primary objectives of this homelab are:

- **Enterprise Networking Lab (CCNA):** Implement physical Link Aggregation (IEEE 802.3ad / LACP) across dual 2.5GbE interfaces, custom Linux bridge hierarchies, and container MACVLAN routing.
- **Data Sovereignty & Media Ingestion:** Replace proprietary cloud photo storage with self-hosted Immich, leveraging Intel QuickSync for hardware-accelerated transcoding and machine learning.
- **Network-Wide Security & DNS:** Run centralized AdGuard Home instances with native Layer 2 visibility via dedicated MACVLAN IP assignment.
- **Local Home Automation:** Host high-availability Home Assistant automation engines decoupled from vendor cloud dependencies.


---

## 2. Hardware Specifications

| Component                   | Specification                                      | Operational Role                           |
| :-------------------------- | :------------------------------------------------- | :----------------------------------------- |
| **Chassis / Host**          | Beelink EQ12                                       | Headless Micro-Server                      |
| **Processor**               | Intel Processor N100 (4C/4T, up to 3.4GHz, 6W TDP) | Alder Lake-N low-power compute             |
| **Graphics**                | Intel UHD Graphics (24 EUs)                        | QuickSync video transcoding & ML inference |
| **Memory**                  | 16GB DDR5 4800MHz SODIMM                           | Unified dynamic host/container memory pool |
| **Boot & Database Storage** | 500GB M.2 2280 PCIe 3.0 NVMe SSD                   | Host OS, LXC rootfs, application DBs       |
| **Bulk Storage (Planned)**  | 2.5" SATA SSD (Empty expansion bay)                | Dedicated media and cold backup volume     |
| **Network Interfaces**      | 2x Intel i225-V / i226-V 2.5GbE RJ45               | LACP EtherChannel bond                     |
| **Upstream Switch/Router**  | TP-Link Archer BE550 (Wi-Fi 7)                     | LACP partner switch & DHCP gateway         |

---

## 3. Proposed Network Architecture & Topology

                  +----------------------------------+
                  |       WAN / Internet Gateway     |
                  +-----------------+----------------+
                                    |
                  +-----------------+----------------+
                  |   TP-Link Archer BE550 Router   |
                  |   (2x 2.5GbE LACP Trunk Group)   |
                  +--------+----------------+--------+
                           | Port 1         | Port 2
                           | (2.5GbE)       | (2.5GbE)
                           +--------+-------+
                                    |
             =======================v=======================
             IEEE 802.3ad Dynamic Link Aggregation (LACP)
             =======================+=======================
                                    |
                  +-----------------+----------------+
                  |      Beelink EQ12 Host (PVE)     |
                  |   Interfaces: enp1s0 + enp2s0    |
                  |   Bond Interface: bond0          |
                  |   Hypervisor Bridge: vmbr0       |
                  +-----------------+----------------+
                                    |
        +---------------------------+---------------------------+
        | (Internal VETH Pair)                                  | (MACVLAN Sub-Interface)
        v                                                       v

+-----------------------+ +-----------------------+
| Debian 12 LXC Host | | AdGuard Home (DNS) |
| IP: 192.168.1.50/24 | | IP: 192.168.1.53/24 |
| - Docker Compose | | (Preserves Client |
| - Immich Stack | | Source IPs) |
| - Home Assistant | +-----------------------+
+-----------------------+
///FIX THIS PART

### Key Networking Design Decisions

1. **IEEE 802.3ad Link Aggregation (LACP):** The dual physical NICs (`enp1s0`, `enp2s0`) will be aggregated into a single logical `bond0` pipe negotiated with the TP-Link BE550. This will yields a 5Gbps aggregate throughput ceiling, multi-client concurrency without buffer bloat, and sub-second link failover.
2. **Layer 3+4 Transmit Hash Policy:** Will configure `xmit_hash_policy layer3+4` in Linux bonding to hash traffic across TCP/UDP ports and IP addresses, ensuring distinct multi-client sessions distribute evenly across both physical links.
3. **MACVLAN for DNS Transparency:** AdGuard Home will run inside a Docker MACVLAN network attached to the host bridge. This will assigns AdGuard its own unique MAC address and local LAN IP, bypassing Docker NAT so client queries log true source IPs and port 53 binds cleanly without host port conflicts.

---

## 4. Software Stack & Virtualization Architecture

- **Hypervisor:** Proxmox VE (Installed with standard `ext4` filesystem to eliminate ZFS ARC memory overhead, preserving the 16GB RAM budget for application workloads).
- **Execution Environment:** Unprivileged Debian 13 LXC Container.
  - `features: nesting=1` (Enables nested namespaces for Docker containerization).
  - `features: keyctl=1` (Allows cryptographic key delegation for Docker's `overlay2` storage driver).

---

## 5. Storage Strategy

This node will implement a three-tier storage lifecycle, physically isolating transactional database IO from heavy sequential media reads:

- **Tier 1 (High IOPS / Transactional):** Internal 500GB NVMe SSD formatted as `ext4`. Houses the Proxmox root partition, LXC root filesystem, Home Assistant SQLite/PostgreSQL databases, and Immich metadata/vector databases.
- **Tier 2 (High-Speed Block / Warm Media):** Future expansion via the internal 2.5" SATA bay. Will be dedicated to raw personal photo libraries (Immich) to ensure rapid thumbnail generation and machine learning inference without USB overhead.
- **Tier 3 (Bulk Sequential / Cold Storage):** Future expansion via an external Multi-Bay USB-C DAS (Direct Attached Storage) enclosure. Bound to the LXC container to host massive static media files (films, TV shows, Proxmox backups). This keeps high-capacity, spinning-rust HDDs physically separate from the compute node, allowing for massive scaling without compromising the mini PC's thermals or NVMe performance.

---

## 6. Service Directory
