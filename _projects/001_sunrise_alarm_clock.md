---
title: "Sunrise alarm clock"
description: "A custom sunrise alarm clock with a little flair"
image: /assets/images/sunrise_alarm/main_image.png
layout: project
permalink: /projects/sunrise-alarm-clock/
date: 22/05/2025
modified_date: 13/04/2025
github_links:
  - url: "https://github.com/elisjackson/sunrise_alarm_pico_2_w_code"
    label: "MicroPython code"
  - url: "https://github.com/.../"
    label: "KiCAD files"
---

**A custom sunrise alarm clock with a little flair, to help me wake up on dark winter mornings. Made in a repeatable way to gift to friends in the same predicament.**

Sunrise alarm clocks emit a gentle light, brightening gradually over a long period of time, emulating the sky's slow transition from night to day. No more sudden awakenings from deep sleep from a clamoring in my ear!

My version would:
- serve as both an alarm clock, and a normal set of lights for my desk area
- have some fun lighting modes, and controllable brightness
- be controllable through my phone or laptop
- be printed on a PCB for some amount of easy manufacturability
- ... with an art silkscreen - wanting it to be on display and looking pretty.

# Prototypes: breadboard to proto PCB

<figure class="project-figure">
  <img src="/assets/images/sunrise_alarm/proto_1.jpg" alt="First prototype">
  <figcaption>Breadboard version - without the second LED string connected</figcaption>
</figure>

Breadboarding started with a spare Pi Pico I had, two NeoPixel LED rings, a logic level shifter (transforming the digital data signals from the Picos' 3.3V to the LEDs' 5V). Buttons would control each LED's 'mode' (Off / Steady white / Raindow dance), and a potentiometer would control all LED's brightness.

Pico code was written in Micropython, and I played around with a few dancing LED ideas to get the hang of coding these.

I had envisaged that the potentiometer might be a good way to control the brightness of the LEDs. However, I found the LEDs would flicker at a fairly high frequency, which was especially visible at low brightness. It worked better as a colour controller.


Plotting the potentiometer voltages, it was clear a noisy signal was the culprit. I hoped that bolting everything down in the proto PCB iteration might help with this - but it didn't. At this point, adding a capacitor to smooth the voltages could have been the thing to do.

Instead I decided to do away with the potentiometer and introduce a third push button. Holding it down would initially *increase* the brightness until I let go. On the next hold the brightness would *decrease*, etc. This worked much better (caveat - after dealing with debouncing - more on that later), being based on a digital signal rather than analogue.

<figure class="project-figure">
  <img src="/assets/images/sunrise_alarm/proto_2.png" alt="Second prototype">
  <figcaption>A more permanent test; proto PCB. The fourth button is to replace one where I'd shorted Pico pins while soldering</figcaption>
</figure>

I also introduced the WiFi connectivity at this stage, using the Pico 2 W as the microcontroller. A HTTP server runs on the Pico, serving a simple webpage. It allows me to control the LED's modes and brightnesses from my devices, but the very key thing here is that this is where I can set the alarm time!

On setting the alarm time, I wanted some feedback from the lights, both to say **"I got the message"**, and to say **"I'll wake you up at x time!"**. Upon receiving the message, the LEDs first light up the alarm hour (if 3am -> light LEDs at the 12, 1, 2 and 3 o'clock positions), followed by the alarm minute (if 45 past -> light LEDs from 12, 1, through to 9 o'clock).

# Final design: the PCB

<figure class="project-figure">
  <img src="/assets/images/sunrise_alarm/kicad.png" alt="KiCAD">
  <figcaption>KiCAD schematic and PCB</figcaption>
</figure>

At this stage, everything was working as I wanted it. The last thing was to design and print it in PCB form - my first foray into PCB design. KiCAD and a fair amount of YouTube got me through the process. 

I'd imagined doing away with the Pi Pico dev board here - and essentially copy-pasting the necessary Pico components (i.e., the RP2350 microcontroller chip, memory, WiFi chip) from the dev board to my new PCB. This looks feasible - there's actually great documentation about it - but I don't currently have the tools to solder such tiny surface-mount components. So yes, I could have the bits and pieces... but they would remain as a bag of bits!

The Pico therefore slots into the PCB. I changed the LEDs connector types so I could solder one end to the PCB, and added a USB-C port for convenient power.

I added a multicolour silkscreen to my PCBs, designed in Photopea. The idea was to illustrate the sun, half-eclipsed and half in its normal glory. Buttons, and their LED pairs, are annotated in a sci-fi-esque font. The printed version turned out OK but lacked some contrast to make the text pop and and the stars stand out. Something to bear in mind next time.

# MicroPython code

### uasyncio

The microcontroller needs to handle multiple tasks simultaneously:
- Serving the webpage
- Listening for button presses & HTTP requests
- Changing the state of each LED independently (e.g. on / off / loop through the rainbow / countdown to the alarm time / gradually increase brightness)

On one core, this can't be done in the traditional synchronous Python regieme. Uasyncio allows the same core to juggle multiple tasks, by introducing asynchronous capability to MicroPython.

In my implementation, the code juggles these tasks by jumping between them. Tasks effectively cycle through being active or sleeping for very short periods.

### Button debouncing

# Learnings & next iterations

### Chips all look the same

Two things here - 
1. Turns out, chips can look identical and have extremely similar names while doing exactly the opposite thing.
For the logic level shifters on my PCBs, I mistakenly ordered the *74AHCT14*, not the *74AHCT125*. It took me so long to realise the problem when my built PCBs didn't work, because they look identical, and I had a massive d'oh moment when I realised my mistake.

    The 125 is the level shifter I wanted. The 14 is is a level shifter - great - **and an inverter**. Ah. It literally flips every 1 and 0.

    In vain I tried correcting by re-wiring (adding wires to account for the pinout differences), and updating the code (inverting the inversion - requesting the opposite colour from the LEDs). It *kinda* worked, but not quite. There's no way to “invert” the protocol cleanly with the code I had, as the data line isn’t just logical bits — it’s tightly timed 0 and 1 pulses. So inverting the signal messes with how the LEDs detect where messages begin and end.

2. Despite the low contrast I'm pleased that I went to the trouble of designing and adding my own artwork. I like the idea of technology being visible, rather than hiding behind a facade. It is minimalism in design. Here, the tech is visible and the tech is aesthetic (so I think, at least!). Plus, it's clearly uniquely mine, rather than just some other PCB.

### Power it properly

USB-C charging is smart - it can deliver a wide voltage and current range. The exact power delievered is negotiated between the device and the charger. If the charger doesn't 'hear from' the device, then the max delivery will be 5V and 1.5A. I wanted my device to take around 48 LEDs in total - meaning I needed more like 3A.

The correct way to negotiate this current would be for the device to literally talk to the charger, continuously. In a USB-C cable (called the 'XXX') there are two wires whose job is to carry these messages back and forth.

In a bit of haste, I decided to shortcut this formal negotiation by wiring 5.1K resistors to ground on the device's XXX pins. This works - 3A can be delivered. The problem is that this misses some of the nuances of negoatiation. If used properly, not only does the device ask, 'can I have 5V and 3A?'; it also asks to the charger, 'are you capable of giving that to me?'. With my resistors, I effectively do the former bu not the latter.

This is fine for chargers that are smart enough - they will just deliver what they can of the requested 3A. However if plugged into a dumber charger, it could overload the charger - which is a bit unsafe. My next iteration would fix this limitation.

### A more suited microcontroller

The Pico is a little overkill, with its XX GPIO pins. A XX with a WiFi chip would be smaller, cheaper, and I'm pretty sure do just as well. Not to mention, fewer headers to solder would have been welcome!

Next time I would also try avoiding using dev boards for the final product, and rather integrating the microcontroller components directly onto my PCB. It wouldn't necessarily make life easier or more convenient, or even save many pennies - but it would make the product more polished, and I'd learn a lot about PCB design and microcntrollers from the process.

### Finishing touches

A few finish touches remain - lenses to diffuse the light, and improving on the current double sided tape (sounds silly, but it does work quite well) as a method for mounting the PCB to a wall or desk.

{% include video.liquid 
  path="https://youtube.com/embed/0EiJWLmmgOM" 
  width="560" 
  height="315" 
  caption="Watch this great video!"
%}



# Final design: the PCB experience

### Learning the ropes

### Artwork

### Python code

### Learnings & next iterations



# Prototype 1 - a working mess

**ANNOTATED IMAGE**

I had a raspberry pi pico laying around, which served as the microcontroller for my initial testing. I had always envisaged that I would use 4 LED rings - one acting as the sunrise alarm near my bed, and three acting as normal lighting in my desk area. Micropython was my language of choice.

For the specialised sunrise ring, I ordered AdaFruit's 12-LED RGBW NeoPixel (I was keen for the RGBW rather than normal RGB version - W for White - as it gives me greater control over the warmth of the light. I have observed that the sun's hue is really quite warm). The three other LED rings could be of the cheaper RGB variety.

For the first tests' power source, I cut one end of a USB -> micro USB cable and distributed the 5V to the Pi Pico and each LED string. The USB end I could plug into any USB socket. In total there are 48 LEDs; each can draw 50mA at max brightness. That's more Amps than many power sources can handle, so I was careful to limit the max LED brightness in the code to a fairly low value. No fires here.

Putting all this together on a breadboard, I ended up with this working mess.

# Prototype 2 - more working less mess

**ANNOTATED IMAGE**

I was keen to tidy things up a bit before moving on to designing the PCB. I also incorporated a logic level shifter at this stage. See, the Pi Pico's output pins sending data to my LEDs operate at 3.3V (that's the "high" signal - the 1's in the binary sequences). However, the LEDs operate at 5V - and they are reading data expecting the "high" signal at that voltage too. While version 1 worked fine, I wanted to create something robust and be sure that data wouldn't be missed by the LEDs due to this discrepancy. So, the logic level shifter takes in the 3.3V data sequence from the Pi Pico, and outputs the same data sequence, but now in 5V - which then goes to the LEDs. Simple.

Prototype 2 was also built on a TThingy. That's tidier. You may also notice that this is no longer a Pi Pico, but a Pico 2 W - which is where the phone/laptop connectivity comes in. On the Pico 2 W is a simple server running, which I can connect to through my local WiFi network. A simple front end now acts as an interface for setting the alarm's time.

# Final design - the PCB experience

### Learning the ropes

### Artwork

### Mistakes

# Next iterations

