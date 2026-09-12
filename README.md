# Multi-Area Network Infrastructure Project

A large-scale enterprise network built in Cisco Packet Tracer, covering multi-area OSPF routing, VTP-managed VLANs, VLSM-based IP addressing, and route summarization. Built as part of a Networking Infrastructure module (scored 95%).

## Overview

The network spans four OSPF areas plus a statically routed segment, connected through a common backbone (Area 0). It was designed to practice real-world enterprise routing and switching concepts:

- **4x OSPF Areas + 1 Static Routing segment**
- **2x independent VTP domains** (VTP A and VTP B) with VLAN propagation
- **VLSM addressing** across 5 IP address pools, sized to requirement (not oversized)
- **Route summarization** at each Area Border Router (ABR) back into Area 0
- **VLAN isolation** — restricted subnets with no inter-VLAN routing (Chemistry, Electronics)

## Topology

![Network Topology](topology-diagram.png)

| Area | Purpose | Key Devices |
|---|---|---|
| Area 0 | OSPF backbone, connects all other areas | Router1–4 (2911s) |
| Area 1 | 3 VLANs (Staff, Students, Admin) on a single switch | 2811 router + L2 switch |
| Area 2 | 3-router hierarchy, 4 switches (Nursing, Engineering, Business, Science) | Router1–3, 4 switches |
| Area 3 | 2 VTP domains (VTP A + VTP B) across 5 switches | Router + 5 switches |
| Static | Single subnet, statically routed into the backbone | 2811 router |

## Addressing Plan

Full VLSM breakdown, including network/broadcast addresses, host ranges, and P2P link addressing, is in [`addressing-plan.xlsx`](./addressing-plan.xlsx).

Address pools used:

| Pool | Assigned To |
|---|---|
| `10.0.0.0/8` | Area 3 (Accounting, Robotics, Electronics, Chemistry, MGMT) |
| `100.64.0.0/10` | Area 1 (Staff, Students, Admin) |
| `172.16.0.0/12` | Area 2 (Nursing, Engineering, Business, Science) |
| `192.168.0.0/16` | Static routing segment |
| `203.0.113.0/24` | All Area 0 point-to-point router links |

Subnets were sized to the exact host-count requirements set out in the brief (e.g. Staff/Students VLANs sized for 400,000 reachable hosts, Admin for 300, Science for 254, etc.), using VLSM to avoid wasting address space while still leaving room for summarization at the ABRs.

## OSPF Design

Each non-backbone area connects to Area 0 through a single ABR, which summarizes its area's subnets into one route:

| ABR | Summary Route Advertised | Covers |
|---|---|---|
| Router1-A1 | `100.64.0.0/10` | All Area 1 subnets |
| Router1-A2 | `172.16.0.0/12` | All Area 2 subnets |
| Router1-A3 | `10.0.0.0/8` | All Area 3 subnets |
| Router1-Static | `192.168.0.0/16` | Static subnet |

This keeps Area 0's routing table lean — it only sees one route per area rather than every individual subnet.

## VLAN & VTP Design

**VTP A** (Server: Switch1-VTPA) — 3 switches, VLANs for Accounting, Chemistry, Electronics, plus a dedicated MGMT VLAN (99) used to manage all switches in the domain. Only the Accounting VLAN has routed network access; Chemistry and Electronics are intentionally isolated so their PCs can only reach others on the same subnet.

**VTP B** (Server: Switch1-VTPB) — 2 switches, single Robotics VLAN spanning both, sized for 4,000 hosts.

## Connectivity Rules Implemented

- PCs with network access can ping each other and router interfaces
- MGMT PC can reach all switches in VTP A
- Chemistry and Electronics PCs are restricted to their own subnet only

## Files in This Repo

| File | Description |
|---|---|
| `network-topology.pkt` | Full Packet Tracer topology file |
| `addressing-plan.xlsx` | Complete VLSM addressing plan (subnets, P2P links, OSPF, VTP, PC configs) |
| `topology-diagram.png` | Visual network diagram |
| `configs/` | CLI `show running-config` exports for each router/switch *(coming soon)* |

