---
publish: "false"
title: "Muscle music"
description: "Generating music from muscle's electrical signals"
projects_page_image: /assets/images/sunrise_alarm/sunrise-alarm-projects-page.png
header_image: /assets/images/sunrise_alarm/main_image.png
layout: project
permalink: /projects/muscle-music/
date: 26/05/2025
modified_date: 26/05/2025
github_links:
  - url: "https://github.com/petra-bachanova/spikerbox_realtime_emg_to_audio/tree/main"
    label: "Python code"
---

**A science day outreach project, encouraging kids to learn about neuroscience by generating music by flexing their muscles.**

{% include video.liquid 
   path="https://www.youtube.com/embed/roLyBcsyGvs?si=yuO3q4sXcNhpY8pf"
   width="560"
   height="315"
%}

This is a long term project that I and *Petra Bachanova (link)* have collaborated on. It is a Python program that builds on the Neuron Spikerbox device developed by *Backyard Brains (link)*. You can try it yourself with some previously recorded muscle signals - check out the GitHub repository!

I should qualify "long term" here. The first iteration made it's "debut" at the 2024 Dartmouth Science Day, with the second used this year at the same event - and we plan to improve it for next year's, too. That's fairly long term by most side project's books. Though, the reality is that it's development has been *highly* concentrated in the takeaway-fuelled hours leading up to each Science Day, plus a little after while things were fresh in our minds. So calling it a long term project is perhaps a little misleading.

The idea here is to build on the Spikerbox, creating a program that helps us to explain to kids (and to many curious parents) how our brains communicate with our muscles.

Here's a brief explainer of what's going on there: ...

The Spikerbox is an ESP32-based device *[is this true?]* which reads electrical signals from electrodes placed on a muscle - a bicep let's say - and:

- amplifies that signal
- "plays" the signal through a little speaker (it sounds like white noise, becoming louder when you contract your bicep)
- converts it to a digital signal
-- and comes with a neat PC program, which plots the signal trace, and extracts the frequencies of the signal

We are replacing that last piece with our own software, with the aim of transforming those signals into music. Music how musical? Well, we won't get a concerto anytime soon - but it's an engaging way to teach kids about their own brains.

# Building blocks

This is a rough sketch of the setup. The backend loops over steps 1-5, until the user sends a frontend command to pause or stop the program.

-- Diagram --

## Backend

### 1: Reading and 'chunking'the data

Two data reading modes are available:

a. Read real-time Spikerbox data over USB serial
b. Read recorded Spikerbox data from file

Either way, the first portion of the backend loop reads a window of data. We've been using a 0.1s window - at the Spikerbox's 10kHz sampling rate, this corresponds to 1000 data points.

Incorporating b. has been been very useful during development. Besides not needing the Spikerbox to hand, it also offers the opportunity for others to easily test the tool.

Beyond this point, the code is agnostic as to whether the data is coming from the Spikerbox or from recorded data - aside from a few places in the front end.

### 2: Filtering

We apply a notch filter at the electricity grid frequency (50 or 60Hz) and its harmonics. The noise contribution from the grid is *extremely* noticable - to the point where we have a garbled mess of data if the Spikerbox is connected to a laptop that is charging from mains.

A band pass filter is also applied - currently set to filter anything outside of 20Hz - 450Hz. We haven't played with this too much, but the values are based on findings from *this study*. We can also see from the spectrogram plots that typically we see the frequencies tail off at around 300Hz, in any case.

-- Study graph --

The plot also shows some other noise contributors: XYZ

### 3 - 4: Get RMS amplitude and play a tone

We use the RMS amplitude as a measure of how hard the muscle is flexing, and thus the pitch of note played. The amplitude is mapped to a note range from C4 (Middle C) to C6 (Middle C plus two octaves). We have a minimum cut-off value, below which no notes are played.

The volume and duration of each note is currently constant (the same as the signal window duration, e.g. 0.1s). The ability to control the note duration would add an interesting dimension to the 'instrument', but we've yet to test ways to achieve this. Some ideas are discussed in the XXX section.

Back to the pitch control - each participant's muscles are different. There is a different amount of baseline noise (which can come from the skin moving over the muscles), and different signal strengths are measured. An athlete has not only larger muscles than a child, but the athlete's brain is able to send more intense **"contract now"** signals to the muscles.

Moreover, each environment is different. Some rooms have a lot of electrical noise. Some tables are near wall plugs, where the 50 or 60Hz of the electricity grid can be picked up by the device. The digital watch of a participant also interferes with the signals picked up by the Spikerbox.

For these reasons the mapping between RMS amplitudes and note pitch is tuned to each participant, which can be done through the front end.

### 5: Get signal frequencies

Inspecting the signal's frequency content currently fuels an academic interest, as it isn't used in the process of making the music.

It is an interesting additional dimension beyond simply "signal strength" in brain-muscle communication. Does it contain useful information for the muscles, or do muslces respond only to the signal strength?

Some research suggests ...

We find the frequencies by doing a STFT. {Elaborate}

In the project's next iteration we may try to use the frequency content to control other elements of the music output, beyond the note's pitch - e.g. volume, tone, the duration of a note - this will need some experimenting.

### Data recording

For consenting partipicants, we record the input data into the program, along with some non-identifiable data such as age band, and any notes about the recording. The notes can be as simple as "worked well on this participant", "noisy signal, maybe from digital watch", or "old electrodes".

This data should help us understand how to improve the recording process - e.g. by showing that we should replace the electrodes more frequently. Additionally, the program can read and process these recordings (just as it would process real-time data from the SpikerBox), which allows us to easily trial code changes.   

## Frontend

The frontend UI is built using the dash (**link**) python library. Data is sent back and forth between the front and backend using socketio (**link**).

Dash was chosen for it's ability to update the quickly UI in real-time, and socketio for its.....

# Learnings


# Future plans





# Prototypes: breadboard to proto PCB

<figure class="project-figure">
  <img src="/assets/images/sunrise_alarm/proto_1.jpg" alt="First prototype">
  <figcaption>Breadboard version - without the second LED string connected</figcaption>
</figure>



<figure class="project-figure">
  <img src="/assets/images/sunrise_alarm/proto_2.png" alt="Second prototype">
  <figcaption>A more permanent test; proto PCB. The fourth button is to replace one where I'd shorted Pico pins while soldering</figcaption>
</figure>



<figure class="project-figure">
  <img src="/assets/images/sunrise_alarm/kicad.png" alt="KiCAD">
  <figcaption>KiCAD schematic and PCB. The thicker traces in the PCB carry the power to the LEDs, via the on/off switch. There are also two traces thicker than most, that carry power from the switch via a diode to the Pico board, which is the large array of pins in the center of the PCB. Most other traces are for data.</figcaption>
</figure>

