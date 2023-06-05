---
title: NVMe and PLX bridges (Chapter 1)
date: 2023-05-26 16:12:40
categories: Homelab
tags: ["homelab", "server stuff", "storage"]
---

{% post_link 'NVMe-and-PLX-bridge-ch1' <img src="https://assets.wflint.me/blog.plx-bridge-header.jpg" width="80%" title="PLX Bridge and Optane" alt="PLX Bridge and Optane"> false %}


<!-- more -->

This [ServeTheHome Thread](https://forums.servethehome.com/index.php?threads/multi-nvme-m-2-u-2-adapters-that-do-not-require-bifurcation.31172/) inventories a few switching card options focusing on those based on PEX and ASM controllers

## Bifurcation vs. Switching?

PCIe bifurcation effectively lets multiple devices share the same physical address, but this is dependent on support both from the motherboard and BIOS.  A PCIe Switch or bridge will occupy the same physical space, but present itself as a single device to the root controller for addressing during initialization.  If the motherboard and bios do not both support bifurcation (giving a heads up that more than one device can be present in that location), the most common result is only the "first" PCIe device will be recognized.

Bifurcation preserves all performance of the connected device as they're configured to directly use the connections as though they were isolated devices.  Switching requires routing traffic first through the switch and _then_ to the target devices.  This does add processing overhead, but has much better compatibility with older (or consumer) boards that lack support for bifurcation.

One huge caveat with these bridges however is looking at the upstream and downstream interfaces and their inherent limitations compared to native connections.

## Performance impacts

For our testing, we'll be using {% post_link 'upgrading-dell-poweredge-r720' the system we upgraded in another post %} (Dell Poweredge R720 with a pair of Xeon E5-2697v2 processors)

These results _will not_ be representative for much outside of this unit, but the combination of lower clock speed on the CPU, PCIe gen3 slots, and slower memory may mean we bottleneck somewhere else in standard-use before the storage.

To be clear, there will be performance left on the table (especially given that we'll be using optane drives for this experiment), but the goal is more to see if sacrificing an extra PCIe x8 slot for a single NVMe drive is a worthwhile tradeoff for the difference in speed or latency 

## Testing!

In addition to our native connection we'll be using two PCIe switches, one based on the [ASM1812](https://www.asmedia.com.tw/product/1e2yQ48sx5HT2pUF/b7FyQBCxz2URbzg0) controller, and one using the successor [ASM2812](https://www.asmedia.com.tw/product/507Yq4asXFzi9qt4/7c5YQ79xz8urEGr1).

This methodology is based on a likeminded setup outlined in a [LevelOneTechs Thread](https://forum.level1techs.com/t/truenas-scale-performance-testing/187486) and all credits go to the author for inspiring this quick set of tests