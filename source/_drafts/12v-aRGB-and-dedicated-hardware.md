---
title: 12v aRGB and Dedicated Hardware
categories: "Neopixel Adventures"
tags: ["homeassistant", "esphome"]
Description: 
---

This is a followup to my previous post on Detolf Lighting which made use of a 5v string of ws8212b LEDs using a generic ESP32 board for use in homeassistant with esphome

Athom dedicated unit

## Benefits
* power budget is
* combined relay and PWM control of output
* integrated design

## Drawbacks
* Higher unit cost

## Side note
* additional relay control over output

## Software implementation
FastLED Clockless is provided by the manufacturer as the LED driver, however it only runs on a pinned (and long deprecated arduino framework).  I've substituted this out in favor of the neopixelbus library and have not run into any issues