---
title: Updating a GMMK2
date: 2023-06-10 17:59:07
categories: "Keyboard"
tags: "keyboard"
description: "I've had a Glorious GMMK2 in storage for a while.  Let's do some basic modifications and see how it behaves!"
---

For the past year or so I've been using a fullsize gen 1 GMMK for work and have recently been looking at trading it out for a new model.  While the GMMK1 is a great value-board, especially in its barebones-chassis configuration coming in around $55, it does lack a few creature comforts (permanently attached cable, no plate foam, proprietary software).  

<!-- more -->

### GMMK2

#### Stabilizers and a Modified "Band-aid" mod

Nothing too fancy, picked up a set of [GSV2 Stabilizers](https://www.gloriousgaming.com/products/gsv2-stabilizers) with the keycaps

One companion to a stabilizer upgrade is to place bandage material under the stabilizer to help mitigate noise.  After looking around I actually had some gaffer tape on hand and substituted that in as it's remarkably similar (if a little less finnicky).

Fairly straightforward without any fuss.  Just remember that there _is_ some z-height added and the stabilizers may not be happy if one side needs to be clipped in (like these are)

#### Keycaps and Modifier Quirks

One of the biggest reasons I've had this board shelved for so long is the fact that the provided 'pudding' style keycaps weren't to my taste and finding replacements has been a major chore.  The GMMK2 is mostly standard, but the right modifier keys (Alt, Function, Ctrl, etc) are all _1u_.  These are almost universally 1.25u keys and seldom come in the narrower variety even in expansion packs.  While you could fill these spots in with custom or artisan caps,  having a uniform look is preferable in my eyes.  

#### Adding O-Rings

This is definitely an optional step, but adding O-rings to the keycaps can help a little to soften feedback.  

I've found it quickest to use a keycap puller to apply them and it really reduces the amount of fumbling around to locate and seat each ring.

#### Spacebar Foam

After getting everything placed I was a little distracted by the spacebar's still decidedly 'hollow' tone, so I took some of the EVA craft foam left over from another project and laid a sheet into the keycap.  It's not a huge difference, but does take the edge off and make the whole thing more pleasant.


#### QMK/VIA (Bonus Round!)

The GMMK2 is advertised as having QMK support, but getting the stars to align in a way that gives a valid firmware flash is not trivial.

Glorious provides an SOP for flashing the board with QMK [here](https://www.gloriousgaming.com/blogs/guides-resources/gmmk-2-qmk-installation-guide), and while it's directionally aligned, the particulars are a bit of a mess.  For one, the board's drivers are nonstandard, so if you install the [official QMK toolbox](https://github.com/qmk/qmk_toolbox) rather than the version compiled by glorious, an external driver installer _will_ be required (potentially with [zadig](https://zadig.akeo.ie/)).

After compiling version specific firmware and pulling the board up in VIA, garbled mapping and an incorrect layout makes it look like something may be pointing to the GMMK Pro rather than the GMMK2 96 ([pro](https://github.com/GloriousThrall/qmk_firmware/blob/54f1adc5dcd308c66d882d9ef041fc5ecb938ba6/keyboards/gmmk/pro/config.h#L24) and [GMMK2](https://github.com/GloriousThrall/qmk_firmware/blob/54f1adc5dcd308c66d882d9ef041fc5ecb938ba6/keyboards/gmmk/gmmk2/config.h#L28) showing same product_id).  Solving for this one is also fairly straightforward though does require trusting VIA's provided firmware.  Downloading from [caniusevia](https://www.caniusevia.com/docs/download_firmware) and flashing with the provided QMK toolbox binary finally took the project across the finish line.