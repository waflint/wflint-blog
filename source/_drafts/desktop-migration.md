---
title: Desktop Migration!
date: 2023-12-01 11:00:08
categories:
tags:
description:
---

This upgrade has been a few months in the making.  My current daily driver desktop is an AM4 based system on an older MSI B550 motherboard, today we're going to upgrade that to X570 and hopefully add some functionality

<!-- more -->



## Testing

### Boot Disk Clone

To validate that nothing would go nuclear at the OS level, I'm testing the windows install by cloning my boot disk and using that on the new board before migrating everything.  I used a [Clonezilla live image](https://clonezilla.org/fine-print-live-doc.php?path=clonezilla-live/doc/03_Disk_to_disk_clone) and everything went smoothly from there.

as a fun extra, I reused this disk to replace another aging SATA drive with a larger capacity.  While normally cloning from a larger drive to a smaller one is not possible, if the donor's data partitions are smaller than the destination (and all near the "beginning" of the disk), it's doable with a few workarounds.

in clonezilla, use advanced mode rather than beginner.  Then when cloning from disk to disk, pick options `-icds` and "Resize partition table proportionally" from the arguments provided.
This copies over the partition table from the source drive as well as the data itself, but skips validation that it'll all fit.

### BIOS tweaks

## **Thunderbolt**

### Multi Monitor?

Short answer: not yet.

While we're able to successfully pass a single display signal through using the DisplayPort connection from the Radeon 6800XT, getting multiple displays out of the dock is still a work in progress.  In theory it should be possible (a previous test with a laptop was able to get multiple out without any issue) as the GPU supports DSC and outputs over DP1.4.



