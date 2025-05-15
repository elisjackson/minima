---
title: "Sunrise alarm clock"
description: "This is my first project."
image: /assets/images/project1.jpg
layout: project
permalink: /projects/sunrise-alarm-clock/
modified_date: 13/04/2025
---

# The why

My alarm clock is getting desperate. Two futile attempts to wake this human, on this December morning, have succeeded only in inducing searching arm movements in its vicinity to snooze it. Attempt three almost succeeds but... nah, actually I'll give myself another 10.

Feeling sympathy for the clock's case, I decided to bench it entirely - to be replaced by a sunrise alarm clock. (This is surely the magic bullet which will dispel those groggy blue winter mornings). What is a sunrise alarm clock? Well, it's a gradually brightening light, helping to induce a more natural awakening than the rude buzzing of my now yeeted standard alarm.

I was initially inspired by this YouTube video; I'll be loosely following Craig's example with a number of my own tweaks. My version would do the following:

- serve as both a sunrise alarm clock, and a normal set of lights for my desk area
- have some fun lighting modes and controllable brightness
- be controllable through my phone or laptop
- be printed on a PCB, with an art silkscreen

This didn't *need* to be printed on a PCB; I'd been wanting to learn how to design PCBs, and this was a good introduction - more on that later.

What is an "art silkscreen" I hear you ask? PCBs are made of multiple layers of 'stuff', including the plastic base, the copper, the mask. Finally, you have the silkscreen layer, which is simply the ink on top used as annotation on the board, e.g. "Resistor 1 goes here, Put Diode 2 This Way Around". Some PCB manufacturers (I used JLBPCB) can print multicolour silkscreens - which opens a world of artistic opportunity that I wanted to make use of. After all, I liked the idea of my PCB being totally visible in my room, rather than squirelled away. I also intended to print a few copies and give sunrise alarm clocks as gifts to some friends and family - and the art is just a nice final touch.

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

# The final version