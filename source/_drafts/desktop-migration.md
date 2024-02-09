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

### 10gig networking

Due to overhead losses from inter-VLAN routing in my unifi environment, this is connected to the NAS subnet.

{% codeblock %}
PS D:\My Documents\Downloads\iperf-3.1.3-win64> .\iperf3.exe -c 10.240.11.118 -p 5201
Connecting to host 10.240.11.118, port 5201
[  4] local 10.240.11.238 port 64342 connected to 10.240.11.118 port 5201
[ ID] Interval           Transfer     Bandwidth
[  4]   0.00-1.00   sec   867 MBytes  7.27 Gbits/sec
[  4]   1.00-2.00   sec  1.01 GBytes  8.69 Gbits/sec
[  4]   2.00-3.01   sec   809 MBytes  6.72 Gbits/sec
[  4]   3.01-4.00   sec   880 MBytes  7.46 Gbits/sec
[  4]   4.00-5.00   sec  1.01 GBytes  8.68 Gbits/sec
[  4]   5.00-6.00   sec   812 MBytes  6.81 Gbits/sec
[  4]   6.00-7.00   sec   998 MBytes  8.37 Gbits/sec
[  4]   7.00-8.00   sec  1.01 GBytes  8.68 Gbits/sec
[  4]   8.00-9.00   sec  1.00 GBytes  8.62 Gbits/sec
[  4]   9.00-10.00  sec  1.00 GBytes  8.62 Gbits/sec
- - - - - - - - - - - - - - - - - - - - - - - - -
[ ID] Interval           Transfer     Bandwidth
[  4]   0.00-10.00  sec  9.30 GBytes  7.99 Gbits/sec                  sender
[  4]   0.00-10.00  sec  9.30 GBytes  7.99 Gbits/sec                  receiver
iperf Done. 
{% endcodeblock %}

Not quite the rated 10gig, but much better than the 2.5gig link I was using in the office.

## **Thunderbolt**

A few quirks I've found with this ASUS motherboard's thunderbolt controller is that it _really_ dislikes hot-plug connections and picking connections back up from sleep.  Non-issue once everything is fully installed and "settled", but definitely a quirk.

To connect the ASUS board's Titan Ridge TB4 port, I inially started with a Belkin TB3 Pro dock ([F4U097tt](https://www.belkin.com/thunderbolt-3-dock-pro/F4U097tt.html)) over a [corning optical TB3 cable](https://www.corning.com/oem-solutions/worldwide/en/home/products-solutions/active-optical-cables/thunderbolt-optical-cables.html).  This functioned more or less as expected once working, but was subject to some gnarly edge cases.  Resuming connections from sleep was a bit hit or miss, but a major issue came from the initial link on boot.  Occasionally it would fail to "stick" and would require a whole dance of physically reseating _both_ ends of the connector and rebooting (rebooting without reseating would never work, so perhaps something in the optical transciever?).

After some additional research and a hint from a [reddit post](https://www.reddit.com/r/Thunderbolt/comments/z33ykc/comment/j3n372s), I picked up a second belkin dock, this time the Thunderbolt 3 Express Dock HD [F4U095](https://www.belkin.com/thunderbolt-3-express-dock-hd---dual-4k-display-85w-psu/P-F4U095.html).  At the time of writing, this dock is discontinued, but is readily available on ebay for less than $50.  This dock has worked flawlessly with the Corning cable.  My only quibble with it is that the onboard displayport handling is only DP1.2, meaning some kind of daisy-chaining was required to get DP1.4 to the monitors.

### Connection hiccups and Cable Quirks

One pitfall with the thunderbolt connection mentioned previously was an issue with hot-plugging not behaving as expected, and after trading out to the second MST hub this still wasn't resolved to an acceptable degree.  What I've gone with longer term is to use hibernation rather than sleep for the system when a low-power standby is needed since I'm not terribly concerned about the wake time so much as that both spin down the onboard HDDs.  From what I've seen, wake-from-sleep frequently causes the dock's thunderbolt connection to be lost entirely (and making it behave as a "generic" USB-C hub) and requires a full reboot to reestablish full thunderbolt connectivity.

Another recurring issue is with the behavior of the corning optical thunderbolt cable itself.  In a windows environment, once working, works exactly as expected.  Unfortunately, this usually involves a few rebots of the dock or computer and reseating it at the host end.  Having to do that on each wake-from-standby or each reboot is not particularly appealing.  Thanks to a [reddit commenter](https://www.reddit.com/r/Thunderbolt/comments/kvuxi3/comment/h2394rb/) suggesting to solve this issue by daisy-chaining it off of another dock, though compatibility of the optical thunderbolt cable itself was also an issue.

I've tried a few additional docks with the results below having tested directly from the PC

### Dock Selections

|     <div align="center">Name</div>                     |        P/N       | Link over optical? | Onboard MST Hub? |                     <div align="center">Notes</div>                                         |
|:-------------------------------------------------------|:----------------:|:------------------:|:----------------:|:--------------------------------------------------------------------------------------------|
| Belkin   Thunderbolt 3 Express Dock HD                 | F4U095           |<font color="green">✔</font>| ❌     | • Links flawlessly including preboot<br>• only passes DP1.2                                  |
| Belkin   Thunderbolt 3 Dock Pro                        | F4U097tt         |<font color="yellow">⚠</font>| ❌    | • Inconsistent link on sleep/wake and reboot<br>• passes DP1.4, but required MST hub         |
| Lenovo   ThinkPad Thunderbolt 3 Dock Gen 2             | 40AN0135US       | ❌     |<font color="green">✔</font>| • did not link over optical<br>• passes DP1.4 when daisy-chained                             |
| HP Thunderbolt Dock 120W G2                            | 2UK37UT#ABA      | ❌     |<font color="green">✔</font>| • host port is accessible from bottom of unit<br>• did not link                              |
| CalDigit Thunderbolt 4 Element   Hub                   | TBT4 Element Hub |<font color="yellow">⚠</font>| ❌    | • updated to FW .40 for better optical compatibility<br>• unit linked, but only passed flickering DP1.2 |
| Dell Thunderbolt 4 Dock                                | WD22TB4          | ❌    |<font color="green">✔</font>| • used USB4-rated "active" coupler<br>• did not link                                         |
|                                                        |                  |                   |                  |                                                                                              |
| Wavlink Thunderbolt Dual DisplayPort Adapter           | WL-UTA21D        |         -         |         -        | Did not work for my application, though not inherently flawed                                |

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
| AW3423DW  | 1440p 100Hz (SDR 8bit) | 1440p 120Hz (**HDR 10bit**) or 175Hz (HDR 8bit) |
| S2719DGF  | 1080p 60Hz (SDR 8bit)  | 1440p 60Hz (SDR 6bit or 8bit)   |

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

## All together

{% mermaid flowchart LR %}
    subgraph basement [Basement]
        I[10gig uplink] <---> PC
        J[POE uplink] <--> kvm["blikvm"]
        kvm <--> s
        subgraph PC [PC]
            s["Asus X570"]
        end
    end
    subgraph office [Office]
        subgraph docks
            PC <--> |"Corning Optical\nThunderbolt 3"|hub1["Belkin\nF4U095"]
            hub1 <-->|"TB3"|hub2["Belkin\nF4U097tt"]
            hub2 --> |"USB-C"|MST["Startech\nMST14CD122DP"]
            hub1 <-->|"USB"|usbhub["USB Hub"]
        end
        subgraph HIDish
            usbhub <--> Mouse/Keyboard
            hub1 <-->|"USB"|monitor1
            MST --> |"DP1.4"|monitor1["Alienware\nAW3423DW"] & monitor2["Dell\nS2719DGF"]
        end
    end
{% endmermaid %}