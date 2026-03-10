# Project Brief

## What I'm building
A DIY music hardware. It takes MIDI commands from a MIDI keyboard or a DAW on a PC, and generates musical notes based on that. The way of making music is with electric motors.


## Current status
Designing proof of concept

## Goals
At the end, we should have an electric design, a software, and a mechanical design. All built and tested. The whole project stored in an organized manner. 


## Main components and structure - general
- Controller: ESP32 S3 devboard
- MIDI interface - currently USB, later MIDI Din connector
- Software - Arduino, developed in VS code + platformio
- Power: battery power

## Main components and structure - music and software
- Mono synth, 1 channel -> later: polyphony, multiple tones
- 1 stepper motor + generic allegro driver
- keep the software modular, so different types of music making methods can be implemented
- simple, object oriented code
  

## Key files / structure
