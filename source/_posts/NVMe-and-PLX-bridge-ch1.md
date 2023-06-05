---
title: NVMe and PLX bridges (Chapter 1)
date: 2023-06-04 18:38:40
categories: Homelab
tags: ["homelab", "server stuff", "storage"]
description: Testing multiple configurations for optane drives in a poweredge r720 TrueNAS Scale ZFS pool where bifurcation is not available.
---

{% post_link 'NVMe-and-PLX-bridge-ch1' <img src="https://assets.wflint.me/blog.plx-bridge-header.jpg" width="80%" title="PLX Bridge and Optane" alt="PLX Bridge and Optane"> false %}


<!-- more -->

One of my current projects involves rebuilding my home storage solution.  As-is, my NAS setup uses a TrueNAS Core virtual machine with an [LSI 9200-8e](https://docs.broadcom.com/doc/12352032) passed through to the VM which is multi-pathed to an MD1200 Powervault.  The pool topology is pretty simple with a 12 disk array in 6 single-mirror VDEVs.  Performance and resiliency has been totally acceptable, but concerns around power draw and lack of meaningful upgrade path (short of playing musical chairs with higher capacity disks) have been simmering for a year or so now.  For the next iteration, I've decided to move from a virtualized solution with external storage to using TrueNAS Scale on bare metal with onboard disks to cut down on as many moving parts as possible (figuratively in terms of hypervisor shenanegains and literally in terms of cabling and rack space)

The crux of this writeup though comes from a combination of a few things.  First, given that I'm planning on occupying a few slots with GPUs and networking cards, space will be at a premium for the optane m.2 NVMe drives I'm planning to use in a metadata VDEV.  These P1600X drives use a [PCIe 3.0 x4 connection](https://www.intel.com/content/www/us/en/products/sku/211867/intel-optane-ssd-p1600x-series-118gb-m-2-80mm-pcie-3-0-x4-3d-xpoint/specifications.html) which would be wasted on the r720's four x8 or two x16 slots (this is the xd config, so we're shy an additional x8 slot).
Compounding this issue, the 12th generation Dell poweredge servers that this r720 is a part of, were the final without PCIe bifurcation; meaning we cannot "split" the slot for direct connectivity.

How then would we go about efficiently making use of both the physical slots and their connectivity?  **_a PLX bridge!_**

Before we begin, I do want to explain that these drives are not intended or expected to be the fastest thing on the block.  They'll be pulling long term duty as metadata storage devices for a new storage pool composed of spinning disks.  All these need to do is accelerate pool operations that HDDs are not optimized for: 

{% blockquote wendell https://forum.level1techs.com/t/zfs-metadata-special-device-z/159954  ZFS Metadata Special Device: Z%}
The ZFS Special Device can store meta data (where files are, allocation tables, etc) AND, optionally, small files up to a user-defined size on this special vdev. This would be a great use-case for solid-state storage.

This will dramatically speed up random I/O and cut, by half, the average number of spinning-rust I/Os needed to find and access a file. Operations like `ls -lr` will be insanely way faster.
{% endblockquote %}


This [ServeTheHome Thread](https://forums.servethehome.com/index.php?threads/multi-nvme-m-2-u-2-adapters-that-do-not-require-bifurcation.31172/) inventories a few switching card options focusing on those based on PEX and ASM controllers

## Bifurcation vs. Switching?

PCIe bifurcation effectively lets multiple devices share the same physical address, but this is dependent on support both from the motherboard and BIOS.  A PCIe Switch or bridge will occupy the same physical space, but present itself as a single device to the root controller for addressing during initialization.  If the motherboard and bios do not both support bifurcation (giving a heads up that more than one device can be present in that location), the most common result is only the "first" PCIe device will be recognized.

Bifurcation preserves all performance of the connected device as they're configured to directly use the connections as though they were isolated devices.  Switching requires routing traffic first through the switch and _then_ to the target devices.  This does add processing overhead, but has much better compatibility with older (or consumer) boards that lack support for bifurcation.

One huge caveat with these bridges however is looking at the upstream and downstream interfaces and their inherent limitations compared to native connections.

## Performance impacts

For our testing, we'll be using {% post_link 'upgrading-dell-poweredge-r720' the system we upgraded in another post %} (Dell Poweredge R720 with a pair of Xeon E5-2697v2 processors)

These results _will not_ be representative for much outside of this unit, but the combination of lower clock speed on the CPU, PCIe gen3 slots, and slower memory may mean we bottleneck somewhere else in standard-use before the storage.

To be clear, there will be performance left on the table (especially given that we'll be using optane drives for this experiment), but the goal is more to see if sacrificing an extra PCIe x8 slot for a single NVMe drive is a worthwhile tradeoff for the difference in speed or latency 

To prep for this, I purchased two PCIe switches, one based on the [ASM1812](https://www.asmedia.com.tw/product/1e2yQ48sx5HT2pUF/b7FyQBCxz2URbzg0) controller, and one using the successor [ASM2812](https://www.asmedia.com.tw/product/507Yq4asXFzi9qt4/7c5YQ79xz8urEGr1).  (un)fortunately the seller I purchased the lower spec card from sent along a second ASM2812 card, so we'll only be testing on that platform.

Looking at the specs, the 2812 cards (despite being physically x8) have a "1, 2, or 4 lane" connection upstream at PCIe gen 3.0 speeds.  Downstream it's capable of exposing 8 lanes at PCIe 3.1 though which should accomodate the optane drives, but this will naturally be limited by the connection to the root device.

## Testing!
This methodology is based on a likeminded setup outlined in a [LevelOneTechs Thread](https://forum.level1techs.com/t/truenas-scale-performance-testing/187486) and all credits go to the author for inspiring this quick set of tests.

we'll be making use of [`fio`](https://github.com/axboe/fio), which is bundled with the TrueNAS Scale version we're using (v22.12.2) using the following parameters:
```
fio --bs=128k --direct=1 --directory=/mnt/foo --gtod_reduce=1 --ioengine=posixaio --iodepth=32 --group_reporting --name=randrw --numjobs=12 --ramp_time=10 --runtime=60 --rw=randrw --size=256M --time_based
```

For pools with multiple drives, I'll be arranging them in single-mirror single VDEV arrangements (or single drive 'control' pools) with stock settings.

one interesting point here too is that while we have 4 optane drives attached, two are direct, and two are switched.  All four drives though are visible "normally"

on a `lspci --vv | grep "Intel Corporation Device 2525"` all devices reported transparently

(truncated version)
```
04:00.0 Non-Volatile memory controller: Intel Corporation Device 2525 (prog-if 02 [NVM Express])
43:00.0 Non-Volatile memory controller: Intel Corporation Device 2525 (prog-if 02 [NVM Express])
46:00.0 Non-Volatile memory controller: Intel Corporation Device 2525 (prog-if 02 [NVM Express])
47:00.0 Non-Volatile memory controller: Intel Corporation Device 2525 (prog-if 02 [NVM Express])
```

for a switched drive: `lspci -vv -s 47:00.0` we even see the expected link state (minus the switch bottleneck)

```
LnkSta: Speed 8GT/s (ok), Width x4 (ok)
```

### Directly attached Pool

The drives in this pool are _not_ making use of the PLX bridge card.  They are on individual carrier boards and are directly connected at full PCIe 3x4 links

{% codeblock mark:3,6,17-18 %}
Jobs: 6 (f=6): [m(1),E(1),m(1),_(1),m(1),_(1),m(1),_(1),m(1),E(2),m(1)][100.0%][r=1204MiB/s,w=1176MiB/s][r=9635,w=9406 IOPS][eta 00m:00s]
randrw: (groupid=0, jobs=12): err= 0: pid=159061: Tue May 30 20:58:43 2023
  read: IOPS=10.0k, BW=1251MiB/s (1311MB/s)(73.3GiB/60044msec)
   bw (  MiB/s): min=   58, max= 2743, per=100.00%, avg=1320.38, stdev=40.79, samples=1358
   iops        : min=  464, max=21945, avg=10562.25, stdev=326.28, samples=1358
  write: IOPS=10.0k, BW=1251MiB/s (1312MB/s)(73.4GiB/60044msec); 0 zone resets
   bw (  MiB/s): min=   49, max= 2702, per=100.00%, avg=1320.30, stdev=40.28, samples=1359
   iops        : min=  396, max=21615, avg=10561.52, stdev=322.21, samples=1359
  cpu          : usr=1.01%, sys=0.18%, ctx=330069, majf=2, minf=7823
  IO depths    : 1=0.1%, 2=0.1%, 4=0.1%, 8=21.2%, 16=54.0%, 32=24.6%, >=64=0.0%
     submit    : 0=0.0%, 4=100.0%, 8=0.0%, 16=0.0%, 32=0.0%, 64=0.0%, >=64=0.0%
     complete  : 0=0.0%, 4=97.2%, 8=0.2%, 16=0.2%, 32=2.4%, 64=0.0%, >=64=0.0%
     issued rwts: total=600540,600804,0,0 short=0,0,0,0 dropped=0,0,0,0
     latency   : target=0, window=0, percentile=100.00%, depth=32

Run status group 0 (all jobs):
   READ: bw=1251MiB/s (1311MB/s), 1251MiB/s-1251MiB/s (1311MB/s-1311MB/s), io=73.3GiB (78.7GB), run=60044-60044msec
  WRITE: bw=1251MiB/s (1312MB/s), 1251MiB/s-1251MiB/s (1312MB/s-1312MB/s), io=73.4GiB (78.8GB), run=60044-60044msec
{% endcodeblock %}

Not a terribly great start, but lets look at a single drive:

{% codeblock mark:3,6,17-18 %}
Jobs: 12 (f=12): [m(12)][100.0%][r=1207MiB/s,w=1180MiB/s][r=9655,w=9439 IOPS][eta 00m:00s]         
randrw: (groupid=0, jobs=12): err= 0: pid=508781: Tue May 30 21:20:26 2023
  read: IOPS=10.3k, BW=1292MiB/s (1355MB/s)(75.8GiB/60041msec)
   bw (  MiB/s): min=  192, max= 2701, per=100.00%, avg=1401.73, stdev=33.87, samples=1320
   iops        : min= 1540, max=21616, avg=11213.05, stdev=270.97, samples=1320
  write: IOPS=10.3k, BW=1292MiB/s (1355MB/s)(75.8GiB/60041msec); 0 zone resets
   bw (  MiB/s): min=  183, max= 2651, per=100.00%, avg=1401.53, stdev=33.19, samples=1320
   iops        : min= 1468, max=21210, avg=11211.45, stdev=265.50, samples=1320
  cpu          : usr=1.22%, sys=0.21%, ctx=329608, majf=0, minf=9986
  IO depths    : 1=0.1%, 2=0.1%, 4=0.1%, 8=21.4%, 16=53.8%, 32=24.7%, >=64=0.0%
     submit    : 0=0.0%, 4=100.0%, 8=0.0%, 16=0.0%, 32=0.0%, 64=0.0%, >=64=0.0%
     complete  : 0=0.0%, 4=97.2%, 8=0.2%, 16=0.2%, 32=2.4%, 64=0.0%, >=64=0.0%
     issued rwts: total=620582,620488,0,0 short=0,0,0,0 dropped=0,0,0,0
     latency   : target=0, window=0, percentile=100.00%, depth=32

Run status group 0 (all jobs):
   READ: bw=1292MiB/s (1355MB/s), 1292MiB/s-1292MiB/s (1355MB/s-1355MB/s), io=75.8GiB (81.4GB), run=60041-60041msec
  WRITE: bw=1292MiB/s (1355MB/s), 1292MiB/s-1292MiB/s (1355MB/s-1355MB/s), io=75.8GiB (81.4GB), run=60041-60041msec
{% endcodeblock %}

### Switch Connected Pool

{% codeblock mark:3,6,17-18 %}
Jobs: 12 (f=12): [m(12)][100.0%][r=951MiB/s,w=948MiB/s][r=7606,w=7580 IOPS][eta 00m:00s]   
randrw: (groupid=0, jobs=12): err= 0: pid=297809: Tue May 30 21:03:42 2023
  read: IOPS=7046, BW=881MiB/s (924MB/s)(51.7GiB/60041msec)
   bw (  KiB/s): min=116423, max=2186240, per=100.00%, avg=1014310.92, stdev=32952.81, samples=1279
   iops        : min=  908, max=17080, avg=7922.96, stdev=257.45, samples=1279
  write: IOPS=7025, BW=879MiB/s (921MB/s)(51.5GiB/60041msec); 0 zone resets
   bw (  KiB/s): min=126163, max=2099557, per=100.00%, avg=1011427.82, stdev=32096.29, samples=1279
   iops        : min=  984, max=16402, avg=7900.54, stdev=250.76, samples=1279
  cpu          : usr=0.82%, sys=0.15%, ctx=223106, majf=0, minf=7445
  IO depths    : 1=0.1%, 2=0.1%, 4=0.1%, 8=21.4%, 16=53.8%, 32=24.7%, >=64=0.0%
     submit    : 0=0.0%, 4=100.0%, 8=0.0%, 16=0.0%, 32=0.0%, 64=0.0%, >=64=0.0%
     complete  : 0=0.0%, 4=97.3%, 8=0.2%, 16=0.1%, 32=2.4%, 64=0.0%, >=64=0.0%
     issued rwts: total=423075,421821,0,0 short=0,0,0,0 dropped=0,0,0,0
     latency   : target=0, window=0, percentile=100.00%, depth=32

Run status group 0 (all jobs):
   READ: bw=881MiB/s (924MB/s), 881MiB/s-881MiB/s (924MB/s-924MB/s), io=51.7GiB (55.5GB), run=60041-60041msec
  WRITE: bw=879MiB/s (921MB/s), 879MiB/s-879MiB/s (921MB/s-921MB/s), io=51.5GiB (55.3GB), run=60041-60041msec
{% endcodeblock %}

around 30% worse, but something else seems off...
a single drive for good measure:

{% codeblock mark:3,6,17-18 %}
Jobs: 12 (f=12): [m(12)][100.0%][r=329MiB/s,w=335MiB/s][r=2629,w=2677 IOPS][eta 00m:00s]         
randrw: (groupid=0, jobs=12): err= 0: pid=416143: Tue May 30 21:15:30 2023
  read: IOPS=8337, BW=1043MiB/s (1093MB/s)(61.1GiB/60045msec)
   bw (  MiB/s): min=   71, max= 2253, per=100.00%, avg=1117.95, stdev=34.42, samples=1332
   iops        : min=  572, max=18018, avg=8942.40, stdev=275.35, samples=1332
  write: IOPS=8330, BW=1042MiB/s (1092MB/s)(61.1GiB/60045msec); 0 zone resets
   bw (  MiB/s): min=   60, max= 2179, per=100.00%, avg=1117.13, stdev=33.80, samples=1332
   iops        : min=  486, max=17430, avg=8935.76, stdev=270.39, samples=1332
  cpu          : usr=1.07%, sys=0.18%, ctx=271835, majf=0, minf=9141
  IO depths    : 1=0.1%, 2=0.1%, 4=0.1%, 8=21.7%, 16=53.5%, 32=24.7%, >=64=0.0%
     submit    : 0=0.0%, 4=100.0%, 8=0.0%, 16=0.0%, 32=0.0%, 64=0.0%, >=64=0.0%
     complete  : 0=0.0%, 4=97.2%, 8=0.2%, 16=0.2%, 32=2.4%, 64=0.0%, >=64=0.0%
     issued rwts: total=500622,500225,0,0 short=0,0,0,0 dropped=0,0,0,0
     latency   : target=0, window=0, percentile=100.00%, depth=32

Run status group 0 (all jobs):
   READ: bw=1043MiB/s (1093MB/s), 1043MiB/s-1043MiB/s (1093MB/s-1093MB/s), io=61.1GiB (65.6GB), run=60045-60045msec
  WRITE: bw=1042MiB/s (1092MB/s), 1042MiB/s-1042MiB/s (1092MB/s-1092MB/s), io=61.1GiB (65.6GB), run=60045-60045msec
{% endcodeblock %}


## Conclusion?

Not looking great in either case...  While these drives are technically only rated for 1760MB/s read and 1050MB/s write, their 4k blocksize IOPS should be closer to 400k and 250k respectively.

This may be an issue with the initial pool configuration or testing at a blocksize of 128k

More to come in chapter two!