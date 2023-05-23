---
title: Upgrading A Poweredge r720
categories: Homelab
tags: ["server stuff", "homelab"]
---

One of my recent projects has been to rehab one of the homelab poweredge boxes to see if I could consolidate workloads from a few standalone devices (and do some testing with TrueNAS Scale on bare metal to stand in for my go-to VMWare and proxmox hypervisors)

My unit came kitted out with an 8x 2.5" drive arrangement, an H710p mini-monolithic drive controller, and a pair of Xeon E5-2690 v0's

We'll be updating our config to include:

* a second 8x 2.5" drive bay
* a replacement backplane to handle the 16 drives
* an H310 mini-monolithic board, flashed into IT mode to act as an HBA
* a pair of Xeon E5-2697 v2's

## Drive Bay and Backplane Installation

## Integrated Storage Controller


## CPU Upgrades