# 06 Minimal Analog Architecture (V1)

This document defines the **least amount of analog circuitry** we can use while letting the **NUCLEO-H723ZG do most of the work** in firmware.

Goal: make the hardware simple, reliable, and universal, so we can use many different passive DD coils by tuning software.

---

## Quick Steps (Simple English)

1) **Standardize the coil connector signals** (not physical pin numbers).
   - TX+, TX-, RX+, RX-, SHIELD_DRAIN

2) **Add only the minimum analog on RX**:
   - protect the inputs
   - bias the signal to mid-supply (so ADC can read it)
   - optional gain using on-chip op-amps (if needed)

3) **Sample RX on the NUCLEO ADC with timer triggering and DMA**.

4) **Do everything else in firmware**:
   - I/Q lock-in demod per frequency
   - filtering, recovery speed
   - baseline subtraction (coil coupling + ground)
   - gain control (AGC)
   - audio output behavior

5) **Start with frequency hopping** (one frequency at a time).
   - easiest to bring up
   - lowest risk of clipping and weird mixing artifacts

---

## Why We Still Need Some Analog

The RX coil produces a small AC signal that can go positive and negative.
The ADC can only read within its voltage range (example: 0 to 3.3V).

So we must:
- protect the connector from ESD and mistakes
- shift the signal to a safe middle voltage (VMID)
- keep the signal linear, meaning no clipping

Everything else can be firmware.

---

## Coil Interface Standard (Logical, Universal)

We are NOT identifying your physical pin numbering yet.
We define a universal logical interface, and later you map your real coil pins to it.

### Connector J1 (COIL_5P)
- J1-1: TX+
- J1-2: TX-
- J1-3: RX+
- J1-4: RX-
- J1-5: SHIELD_DRAIN

### Shield rule
- Connect SHIELD_DRAIN at one point only (control box end).
- Do not connect shield at both ends, it can create noise loops.

---

## Minimal RX Hardware Blocks (V1)

### Block 1: RX Input Protection (Mandatory)
Purpose: protect the MCU and analog circuitry.

Recommended parts:
- series resistor on RX+ and RX- (one each)
- low-capacitance ESD protection device on RX lines

Notes:
- Keep protection capacitance low so we do not load the coil.
- Place this close to the coil connector.

### Block 2: Mid Bias (VMID) Generator (Mandatory)
Purpose: shift AC signals so the ADC can read them.

VMID is approximately half of the analog supply:
- if analog supply is 3.3V, VMID is about 1.65V

Implementation options:
- resistor divider + filter caps, then buffer VMID
- DAC output set to midscale, then buffer VMID

Buffering VMID is strongly recommended because it keeps the bias stable.

### Block 3: Gain Stage (Optional, but likely needed)
Purpose: bring RX signal into a usable range without clipping.

V0.1 option (absolute minimum):
- no gain stage
- just bias and sample
- rely on longer I/Q integration in firmware

V1 recommended option (still minimal):
- use the MCU internal op-amps as simple non-inverting amplifiers
- one op-amp for RX+
- one op-amp for RX-
- sample both legs and subtract digitally

This keeps the BOM small and keeps gain steps software controlled.

### Block 4: Anti-alias Filtering (Optional footprints)
Purpose: reduce high-frequency trash from getting into the ADC and aliasing down.

We will add footprints for small RC filtering, but start conservative:
- we can DNP these parts until needed
- if EMI is bad, we populate them

---

## Recommended RX Signal Flow

We sample RX+ and RX- separately, then subtract in firmware.

This keeps analog simple, and still gives good noise rejection.

### Signal flow
RX+ and RX- from coil:
1) protection (series R + ESD)
2) bias to VMID
3) optional internal op-amp gain per leg
4) ADC measures both channels
5) firmware computes:
   - RX_DIFF = RX+ - RX-

Then lock-in happens on RX_DIFF.

---

## TX Hardware Notes (Minimal Approach)

TX still needs a real driver. The MCU cannot drive the coil directly.

For V0.1 bring-up:
- allow external TX drive through a header
- use low power first, then scale up later

For V0.2:
- add MOSFET driver + MOSFETs + protection
- add current sense if possible (huge for safety and coil flexibility)

---

## Firmware Responsibilities (The "Most Going Through The Nucleo" Part)

### 1) Timing and synchronization
- one master timer controls:
  - TX waveform timing
  - ADC sample timing
- ADC conversions triggered by timer
- DMA double buffer to capture samples reliably

### 2) Lock-in I/Q demod
For each frequency f:
- multiply samples by sin and cos references at f
- low-pass filter to extract I and Q
- compute magnitude and phase from I/Q

### 3) Baseline subtraction
The RX coil always sees large TX coupling and ground response.
We subtract a baseline:
- baseline per frequency for I and Q
- update slowly in tracking mode

### 4) Gain control (AGC)
We must prevent clipping:
- detect ADC peak values
- if near rail, reduce gain or reduce TX power
- if too small, increase gain

### 5) Audio behavior
The “feel” of the detector is mostly audio:
- threshold hum
- target tone beeps
- modulation and smoothing (recovery speed)

---

## What Goes Into EasyEDA (Minimal Analog Version)

### Sheet 1 (Do firs
