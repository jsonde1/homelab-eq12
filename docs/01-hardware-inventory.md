# Hardware & Network Inventory

This document serves as the master record for the physical hardware configuration, interface mapping, and static IP allocations for the Beelink EQ12 homelab node.

> **Security Note:** Physical MAC addresses have been obfuscated in this public repository.

## 1. Host Node: Beelink EQ12

| Component               | Specification             | Notes                                          |
| :---------------------- | :------------------------ | :--------------------------------------------- |
| **Model**               | Beelink EQ12              | Alder Lake-N Micro-PC                          |
| **CPU**                 | Intel Processor N100      | 4 Cores, 4 Threads, 6W TDP, QuickSync          |
| **RAM**                 | 16GB Crucial DDR5 4800MHz | Single SODIMM                                  |
| **Storage (Primary)**   | 500GB NVMe M.2 2280       | OS, LXC, Databases (`ext4`)                    |
| **Storage (Expansion)** | _Empty_                   | 2.5" SATA Bay (Reserved for future media pool) |

## 2. Physical Network Interfaces & Link Aggregation

> **Note:** Proxmox Network Interface Pinning is active.

The host will utilise dual 2.5 Gigabit Ethernet ports bonded via IEEE 802.3ad (LACP).

| Interface  | Hardware ID / MAC   | Role                | Connected To            |
| :--------- | :------------------ | :------------------ | :---------------------- |
| **`nic1`** | `XX:XX:XX:XX:XX:FC` | LACP Bond Slave 1   | TP-Link BE550 (Port 1)  |
| **`nic2`** | `XX:XX:XX:XX:XX:FD` | LACP Bond Slave 2   | TP-Link BE550 (Port 2)  |
| **`TBC`**  | `XX:XX:XX:XX:XX:XX` | LACP Master (5Gbps) | Aggregated Logical Link |

## 3. IP Address Allocation Scheme

The local subnet (`192.168.0.0/24`) is strictly segmented to prevent IP collisions between dynamic client devices and static infrastructure.

### DHCP Configuration (TP-Link BE550)

- **Subnet Mask:** `255.255.255.0`
- **Gateway:** `192.168.0.1`
- **Dynamic DHCP Pool:** `192.168.0.2` – `192.168.0.199` (IoT, Phones, Laptops)
- **Static Infrastructure Block:** `192.168.0.200` – `192.168.0.253` (54 Reserved IPs)

### Static Infrastructure Assignments

| IP Address      | Hostname / Service | Network Type | Function                       |
| :-------------- | :----------------- | :----------- | :----------------------------- |
| `192.168.0.200` | `pve-node-01`      | Host         | Proxmox VE Hypervisor UI / SSH |
