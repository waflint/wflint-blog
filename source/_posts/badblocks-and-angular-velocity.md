---
title: badblocks and Effective I/O
date: 2023-07-01 10:16:39
categories: "Homelab"
tags: ["storage", "server stuff", "homelab"]
description: Quick post about hard disk behavior
photo: "https://assets.wflint.me/blog.badblocksdiskrate.png"
---

<!-- more -->

Quick post today about a fun observation!

I've been running [badblocks](https://en.wikipedia.org/wiki/Badblocks) scans on some new disks and noticed that over the 26+ hour runtime that the effective data rate of the run was gradually decreasing over time.  This seemed a little odd given that disks are assumed to have consistent IOPS, but this workload shows off a fun side of 'rated' vs 'effective' performance.

We initiate the run using the following command inside a TMUX session and let it run for the next day:
`sudo badblocks -t random -w -s -b 4096 /dev/sdd`
_specifying block size is required for these larger disks as detailed in [this blog post by Dave Jansen](https://www.davejansen.com/use-badblocks-to-check-larger-hard-drives/)_

Since badblocks works through each block on the disk, its progression from the outside of the platter to the inside is a great demonstration of how a drive's effective performance could change dramatically based on what the physical location of its data!  While the spindle speed remains constant, the amount of the disk rotated under the drive head is substantially higher at the outside of the platter than later in the run when the head is near the center of disk. 