# docs/02_requirements.md

# Requirements (V1)

## Core goal
Build an open-source multi-frequency VLF detector platform using a passive 5-pin DD coil and an STM32 NUCLEO-H723ZG, with minimal analog and DSP-heavy firmware.

## Coil
- Type: passive DD
- Connector: 5-pin
- Pins: TX+, TX-, RX+, RX-, Shield/Drain (shield connected at ONE point only)

## Frequencies (V1)
- Mode: frequency hopping (one frequency at a time)
- Default set (editable later): 5 kHz, 10 kHz, 15 kHz, 20 kHz
- Dwell per frequency: 5 ms (target), configurable 2–10 ms

## Sampling
- ADC sampling must be timer-triggered and phase coherent to TX timing
- DMA double buffering required
- Initial target sample rate: 200 kS/s (tune later)

## RX chain requirements
- Must never clip during worst-case TX-to-RX coupling
- Must have 2–4 gain steps (jumper or switch IC, firmware controlled later)
- Must have mid-bias reference and filtering
- Must include input protection at the coil connector
- Must include test points: RX_IN+, RX_IN-, BIAS, RX_POSTGAIN, 3V3A, GND

## TX bring-up strategy
- V0.1: allow external low-power TX drive (bench/function gen/amplifier) for safe testing
- V0.2: add on-board TX driver with current limiting and protection

## Success criteria (V1)
- Stable single-frequency lock-in I/Q
- 4-frequency hopping I/Q stable in air and in soil
- Ground baseline subtraction works (manual GB + basic tracking)
- Usable audio response outdoors

