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

A few quirks I've found with this ASUS motherboard's thunderbolt controller is that it _really_ dislikes hot-plug connections.  Non-issue once everything is fully installed, but definitely a quirk.

For this deployment, the ASUS board's Titan Ridge TB4 connection will be sent to a Belkin TB3 dock ([F4U097tt](https://www.belkin.com/thunderbolt-3-dock-pro/F4U097tt.html)) over a [corning optical TB3 cable](https://www.corning.com/oem-solutions/worldwide/en/home/products-solutions/active-optical-cables/thunderbolt-optical-cables.html)

### Multi Monitor?

Short answer: ~~not yet.~~ yes*

While we're able to successfully pass a single display signal through using the DisplayPort connection from the Radeon 6800XT, getting multiple displays out of the dock is still a work in progress.  In theory it should be possible (a previous test with a laptop was able to get multiple out without any issue) as the GPU supports DSC and outputs over DP1.4.

without a native solution on-dock (one monitor over displayport, one over the downstream thunderbolt port), it's necessary to use an MST hub.  To preface, I have a dual monitor setup that run both at 1440p and 144hz (one Dell S2719DGF and one Alienware AW3423DW).  The alienware's QD-OLED panel really shines in HDR and is one of the primary reasons to have it, so support for HDR at all is a big plus.  Support for 10bit color depth is something I've also been keen on as it practically eliminates banding in media playback.

My first try with this was using a startech DP1.4 hub (PN:[MST14DP122DP](https://www.startech.com/en-us/display-video-adapters/mst14dp122dp)), and while it did work, did not support DSC and delivered a less-than-ideal experience. HDR was out from the beginning, and if I wanted to keep 10 bit color on the alienware monitor I was locked to 100hz on it and 60 on the secondary monitor.  Bumping down to 8bit color allowed up to 100hz, but that wasn't a stellar consolation prize.

Next up was trading out to a different MST hub.  This time a startech thunderbolt/USB-C to DP1.4 adapter (PN:[MST14CD122DP](https://www.startech.com/en-us/display-video-adapters/mst14cd122dp)).  From the start, much better options with 1440p immediately displaying on both monitors and HDR enabled on the primary.  On further investigation, not only was HDR and 10bit color available on the alienware monitor, but the full refresh-rate of 175Hz was also enabled.  Interestingly enough, I did also find that the Dell monitor (via the Radeon control panel) would accept 6bit color depth.  While we're not bottlenecked on bandwidth at this point, knocking that down a level gives us additional headroom and did not cause any noticable degredadation.

In both cases however, support for variable refresh rate (VRR/Freesync/G-Sync) was unavailable.

results:
|           | MST14DP122DP           | MST14CD122DP            |
|-----------|------------------------|-------------------------|
| Alienware | 1440p 100Hz (SDR 8bit) | 1440p 120Hz (HDR 10bit) |
| Dell      | 1080p 60Hz (SDR 8bit)  | 1440p 60Hz (SDR 6bit)   |

### Connection hiccups

One pitfall with the thunderbolt connection mentioned previously was an issue with hot-plugging not behaving as expected, and after trading out to the second MST hub this still wasn't resolved to an acceptable degree.  What I've gone with longer term is to use hibernation rather than sleep for the system when a low-power standby is needed since I'm not terribly concerned about the wake time so much as that both spin down the onboard HDDs.  From what I've seen, wake-from-sleep frequently causes the dock's thunderbolt connection to be lost entirely (and making it behave as a "generic" USB-C hub) and requires a full reboot to reestablish full thunderbolt connectivity.  Hibernation seems to avoid this with the caveat of sometimes requiring a reseat of the MST hub if HDR isn't being passed through properly.

## Remote management

The final piece of this puzzle comes from a piKVM card.

to mitigate some of the hot-plug issues with the thunderbolt setup, I'll be using the [piKVM's API](https://docs.pikvm.org/api/) for remote state control (as well as offering some redundancy for lights out-management)

while it's platform-agnostic, I'll be using Home Assistant's [REST command integration](https://www.home-assistant.io/integrations/rest_command/) to interact with the service

{% codeblock lang:yaml %}
rest_command:
  my_request:
    url: 'https://pikvm.localdomain.net/api/atx/click?button={% raw %}{{ button }}{% endraw %}'
    method: POST
    headers:
      X-KVMD-User: !secret pikvm_user
      X-KVMD-Passwd: !secret pikvm_pass
{% endcodeblock %}

(note that the above user needs to be configured as a KVM/interactive user rather than a SSH/system user)