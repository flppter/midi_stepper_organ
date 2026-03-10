# MIDI Protocol Summary
*Relevant scope: USB-MIDI, monophonic note playback on ESP32*

---

## What is MIDI

MIDI (Musical Instrument Digital Interface) is a serial communication protocol. It transmits *events* (things that happen), not audio. A MIDI message says "note X started" or "note X stopped" — the receiving device decides what to do with that.

Messages are binary, 1–3 bytes each, transmitted at 31250 baud (classic DIN). Over USB-MIDI, the same messages are wrapped in 4-byte USB packets, but the content is identical.

---

## Message Structure

Every MIDI message starts with a **status byte**, followed by 0–2 **data bytes**.

The status byte has two parts packed into one byte:
- High nibble (bits 7–4): **message type**
- Low nibble (bits 3–0): **channel number** (0–15, representing channels 1–16)

Data bytes always have bit 7 = 0, so their value is 0–127.

```
Status byte:  [1 t t t | c c c c]
               ^         ^
               type      channel (0-15)
```

---

## The Three Messages That Matter for This Project

### 1. Note On — `0x9n`
Sent when a key is pressed.

| Byte | Value | Meaning |
|------|-------|---------|
| Status | `0x90` – `0x9F` | Note On, channel 1–16 |
| Data 1 | 0–127 | Note number |
| Data 2 | 1–127 | Velocity (how hard the key was pressed) |

> **Special case:** Note On with velocity = 0 is treated as a Note Off. Many devices send this instead of a real Note Off message. You must handle this.

### 2. Note Off — `0x8n`
Sent when a key is released.

| Byte | Value | Meaning |
|------|-------|---------|
| Status | `0x80` – `0x8F` | Note Off, channel 1–16 |
| Data 1 | 0–127 | Note number |
| Data 2 | 0–127 | Release velocity (usually ignorable) |

### 3. Control Change — `0xBn` *(low priority for now)*
Sends controller data (mod wheel, sustain pedal, volume, etc.). Not needed for basic note playback, but worth knowing it exists so you can ignore it safely.

---

## MIDI Note Numbers

Note numbers run 0–127. Middle C is note **60**, defined as C4.

The formula to convert a note number to frequency:

```
f = 440 * 2^((n - 69) / 12)
```

Where `n` is the note number and 440 Hz is A4 (note 69).

Selected reference points:

| Note | Number | Frequency |
|------|--------|-----------|
| C4 (Middle C) | 60 | 261.63 Hz |
| A4 | 69 | 440.00 Hz |
| A3 | 57 | 220.00 Hz |
| A5 | 81 | 880.00 Hz |

The full piano range is roughly notes 21–108 (A0 to C8), frequencies 27.5 Hz to 4186 Hz.

---

## Channels

MIDI has 16 channels (1–16), encoded as 0–15 in the status byte. Channels let multiple instruments share one cable.

**Channel 10** is conventionally reserved for drums/percussion.

For this project (mono synth): listen on **channel 1** only, or listen on all channels — both are fine. Filtering by channel is simple but optional at this stage.

---

## USB-MIDI Packet Format

Over USB, each MIDI message is padded into a **4-byte packet**:

```
Byte 0: Cable number (high nibble) + Code Index Number / CIN (low nibble)
Byte 1: Status byte
Byte 2: Data byte 1
Byte 3: Data byte 2
```

The CIN indicates the message type and is derived from the status byte. In practice, most USB-MIDI libraries (like the Arduino `USB-MIDI` library) handle this unpacking for you — you receive already-parsed note number and velocity values.

---

## Minimum Implementation Logic

```
on receiving a message:
    if Note On AND velocity > 0:
        start playing note (note number → frequency → drive motor)
    if Note Off OR (Note On AND velocity == 0):
        stop playing
```

That is sufficient for a functional mono synth.
