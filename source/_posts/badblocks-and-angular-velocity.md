---
title: badblocks and Effective I/O
date: 2023-07-01 10:16:39
categories: "Homelab"
tags: ["storage", "server stuff", "homelab"]
description: Quick post about hard disk behavior
---

{% post_link 'badblocks-and-angular-velocity' <img src="https://assets.wflint.me/blog.badblocksdiskrate.png" width="80%" title="Disk Activity Curves" alt="Disk Activity Curves"> false %}

<!-- more -->

Quick post today about a fun observation!

I've been running [badblocks](https://en.wikipedia.org/wiki/Badblocks) scans on some new disks and noticed that over the 26+ hour runtime that the effective data rate of the run was gradually decreasing over time.  This seemed a little odd given that disks are assumed to have consistent IOPS, but this workload shows off a fun side of 'rated' vs 'effective' performance.

Since badblocks works through each block on the disk, its progression from the outside of the platter to the inside is a great demonstration of how a drive's effective performance could change dramatically based on what the physical location of its data!