---
title: Detolf lighting with WS2812 and Homeassistant
date: 2022-09-18 16:12:40
tags: ["detolf", "esphome", "homeassistant", "neopixel", "WS2812"]
categories: "Neopixel Adventures"
---

{% post_link 'detolf-lighting-with-ws2812-and-homeassistant' <img src="https://assets.wflint.me/blog.scaledoutcome.jpg" width="80%" title="Neopixel Detolf Lighting" alt="Neopixel Detolf Lighting"> false %}

<!-- more -->

## Preface

We have a few Ikea detolfs around the house and have had issues lighting them in a satisfactory manner. 

Initially I used the “stock” option of sticking 3 ledberg spotlights at the top of the case and feeding power through the lid, but this doesn’t provide even lighting through the layers, and left power leads dangling off the case.

{% image https://assets.wflint.me/blog.scaledlookup.jpg 750 350 %}

I iterated over a few revisions which used 5v LED tape soldered to USB leads, but still had a few issues.  The first pass used a zig-zag ladder pattern along the front which was good for spreading light between the layers, but was very visible and didn’t inject much more light overall.  Further, the passive USB-based input was definitely not the right tool for the job once you get past about a meter of tape.

The second pass, while still passively controlled, wrapped the tape around the edges of the case which significantly increased the surface area of light emitted while also not cutting into the view of the case itself. As this introduced right-angles into the equation, the tape needed to be cut and rotated with the use of corner-addons (more on this later)

<div style="width:80%; margin:auto;">

{% grouppicture 2-2 %}
  {% img https://assets.wflint.me/blog.scaledladder.jpg '"laddered style" "laddered style"'%}
  {% img https://assets.wflint.me/blog.scaledperimiter.jpg '"perimiter style" "perimiter style"'%}
{% endgrouppicture %}

</div>
<p style="text-align:center"><sup><em>conveniently, the perimiter path is around 14 feet, putting it comfortably within the standard 5-meter LED tape size!</sup></em></p>


While working through it though I realized I wanted greater control and functionality out of the whole setup. With passive control we’re locked into a single hue and on/off functionality, but by moving to a WS8212 and smart guts, I could stick with the same form factor, but improve everything else with the power of an [arduino board](https://smile.amazon.com/gp/product/B0B19KRPRC) and home automation.

## The Corners

The most involved part of this project was managing the four required 90° turns.

Initially I made use of the bulk-pack alligator clip ones, however between the inconsistent contact and greatly accelerated power loss between segments made them more trouble than they were worth.

After doing some research, I came across a great [writeup by Josh Levine](https://wp.josh.com/2015/09/12/neopixel-corner-cases-accurate-and-easy-rectangles-with-ws2812b-strips/) tackling this exact issue that set down a board design (which looks to have hit some level of indie manufacture in the time since). As you can see, we did make some slight modifications, but otherwise the design is a real winner!  We opted for 3rd party fabrication rather than etching the boards in-house though as that'll be reserved for a future project. The design itself accommodates the pixels being soldered for “left” and “right” bends, but make sure to figure out which and how many you need before soldering things down. In my case I needed 4 “left” bends as the tape progresses in a spiral pattern. Populating the pixels can either be done by cannibalizing extras from the strip or ordering extras. I went with a [batch from adafruit](https://www.adafruit.com/product/3094) and they’ve been great.

{% youtube rjnrC6yzgLI %}

## Power

Rule of thumb I’ve seen for WS8212B/SK6812 is to budget approximately 50mA per pixel. In this instance, a standard 30/m string for a 5m length sums up to 5v at 7.5A, so I rounded up to a 10A supply.

Be aware that with cheap power supplies, you get what you pay for. From reviews and from experience, drive the average supply anywhere near its rated amperage and voltage sags will not be far behind. This is a twin peril with the “helpful” barrel breakout jacks that may be included with the purchase.

<p style="text-align:center"> {% image https://assets.wflint.me/blog.scaledbreakout.png '"do not trust breakout jacks" "do not trust breakout jacks"'%}</p>

Typically these are thrown in with the [“normal”](https://smile.amazon.com/gp/product/B07CMM2BBR) 5v-laptop-brick types of supplies, but please do yourself a favor and spend a few bucks on ones that won’t burn your house down. The high resistance (and iffy internal contacts) can wreak havoc on stability and the power spent dumping waste heat can easily eat whatever headroom is left in the power budget. Save yourself the headache and spend a few extra dollars on a purpose built socket (like [this](https://smile.amazon.com/gp/product/B09WJC414P)).

I’ve found that at this length of tape, powering the strip from both ends is warranted. powering from only one side will technically work, but you will see drop-off over the length of the strip.

## The Code

One of the simplest portions of this whole rigamarole was integrating with [Home Assistant](https://www.home-assistant.io/).

Using ESPHome’s [Neopixelbus platform](https://esphome.io/components/light/neopixelbus.html?highlight=ws2812x), things went very smoothly

One element that wasn’t self-evident but turned out to be a great addition was bespoke color presets. Using an online [color temperature calculator](https://academo.org/demos/colour-temperature-relationship/).  By taking our nominal channel brightness of 255 and dividing each component into it, we get the proportion of each channel needed for a particular temperature.  This means we get a stable hue with brightness control without any additional fuss.

{% grouppicture 3-3 %}
  {% img https://assets.wflint.me/blog.scaled3000k.jpg '"3000k" "3000k"'%}
  {% img https://assets.wflint.me/blog.scaled4500k.jpg '"4500k" "4500k"'%}
  {% img https://assets.wflint.me/blog.scaled6000k.jpg '"6000k" "6000k"'%}
{% endgrouppicture %}

The operative part of the codeblock is pasted below:

{% codeblock lang:yaml %}
light:
  - platform: neopixelbus
    type: grb
    pin: GPIO13
    num_leds: 150
    name: "Office Detolf LEDs"
    id: "Office_Detolf_LEDs"
    variant: ws2812x
    default_transition_length: 1s
    effects: 
      - addressable_rainbow:
      - addressable_random_twinkle:
          twinkle_probability: 7%
      - addressable_color_wipe:
      - addressable_scan:
      - automation:
          name: Warm White 3000K
          sequence:
            - light.addressable_set:
                id: "Office_Detolf_LEDs"
                red: 100%
                green: 69.4%
                blue: 43.1%
      - automation:
          name: Cool White 4500K
          sequence:
            - light.addressable_set:
                id: "Office_Detolf_LEDs"
                red: 100%
                green: 85.5%
                blue: 73.3%
      - automation:
          name: Cool White 6000K
          sequence:
            - light.addressable_set:
                id: "Office_Detolf_LEDs"
                red: 100%
                green: 96.5%
                blue: 92.9%  
{% endcodeblock %}

## The End Result!

{% youtube k2BvnwIfmRQ %}