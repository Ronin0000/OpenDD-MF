# OpenDD-MF (Open Double-D Multi-Frequency VLF)

OpenDD-MF is an open-source **multi-frequency VLF metal detector platform** built around a **passive 5-pin DD search coil** and an **STM32 NUCLEO-H723ZG**.  
The core goal is a detector that stays **hardware-simple and reliable**, while most behavior is **defined in firmware** (digital I/Q lock-in, ground balance, filtering, audio response, profiles).

> Status: **V1 in progress** (project scaffolding + coil/interface definition)

---

## Why this exists

Most DIY detectors fail because the analog path is fragile (clipping, drift, layout sensitivity). OpenDD-MF is designed to:
- keep analog circuitry **minimal, linear, and hard to break**
- do the “smart” parts in software (multi-frequency processing, phase/magnitude features, ground subtraction)
- produce a real **build manual**, test protocol, and repeatable results

---

## V1 Goals

### Required (V1)
- Passive **DD coil (5-pin)**: TX pair, RX pair, shield/drain
- **4-frequency** operation (start with **frequency hopping**)
- **Digital lock-in (I/Q) demodulation** per frequency
- **Ground balance** (manual + basic tracking)
- Stable **audio response** (threshold + target tones)
- Simple UI (start with serial; OLED optional later)
- A build doc that is repeatable: wiring, testing, calibration, troubleshooting

### Not required (V1)
- Perfect target ID matching commercial detectors
- True simultaneous multi-tone TX on day 1
- Waterproof enclosure on day 1
- Full “product UI” on day 1

---

## Hardware Philosophy (keep it simple, keep it stable)

### What we keep in analog
The minimum set of analog blocks needed to safely sample the coil:
- input protection (ESD, clamps, series resistors)
- one clean gain stage (prefer differential)
- gentle anti-alias filtering
- mid-biasing into ADC range
- 2–4 gain steps (GPIO controlled), so firmware can prevent clipping

### What we do in firmware
- lock-in I/Q demod
- filtering and recovery speed tuning
- baseline subtraction (ground + coupling)
- gain control strategy
- detection decision logic
- audio tone generation and modulation
- profiles (Park/Field/Beach-style behavior later)

---

## System Overview

### Coil (Passive DD, 5-pin)
Typical 5 pins:
- TX+ / TX-
- RX+ / RX-
- Shield/drain (connected at ONE point only)

**First milestone:** confirm coil pinout with a multimeter resistance table.

### TX (Transmit)
- Start with **frequency hopping** for maximum stability
- Firmware controls: frequency list, dwell time, TX amplitude
- Hardware provides: robust driver, current control, protection

### RX (Receive)
- RX coil signal is sampled by ADC using timer-triggered conversions + DMA
- Firmware extracts magnitude + phase per frequency using I/Q lock-in

---

## Milestones (V1)

1. **Repo + docs skeleton complete**
2. Coil pinout confirmed (TX pair, RX pair, shield)
3. TX bring-up at low power verified on scope
4. RX AFE bring-up (bias correct, no clipping) + noise floor measured
5. Single-frequency lock-in works
6. 2-frequency hopping works
7. 4-frequency hopping works
8. Ground balance (manual + tracking) behaves in real soil
9. Usable audio response outdoors (repeatable tests)

---

## Repository Layout

### Docs
- `docs/00_project_overview.md`  
  Goals, milestones, architecture notes.
- `docs/01_coil_pinout.md`  
  Coil mapping (pin numbering + resistance table).
- `docs/02_requirements.md`  
  Frequency set, sample rate targets, performance targets.
- `docs/03_bringup_plan.md`  
  Safe bring-up steps (no smoke), scope checks, clip checks.
- `docs/04_test_protocol.md`  
  Air tests, ground tests, EMI tests, logging format.
- `docs/05_troubleshooting.md`  
  Symptom → cause → fix.

### Hardware (EasyEDA)
- `hardware/easyeda/README.md`  
  Project link and export/versioning rules.
- `hardware/easyeda/exports/schematics_pdf/`  
  Schematic PDFs by version.
- `hardware/easyeda/exports/gerbers/`  
  Fabrication outputs.
- `hardware/easyeda/exports/bom/`  
  BOM exports.
- `hardware/easyeda/exports/pick_and_place/`  
  PnP files (if used).
- `hardware/easyeda/libraries/`  
  Custom symbols/footprints (if needed).

### Firmware
- `firmware/README.md`  
  Toolchain and build notes.
- `firmware/src/`  
  Main application.
- `firmware/include/`  
  Configuration and shared headers.
- `firmware/tools/`  
  Scripts for log parsing and offline analysis (optional).

### BOM
- `bom/bom_v1.md`  
  Draft BOM organized by subsystem.

---

## Getting Started (What to do first)

### Step 1: Map the 5-pin coil
Open `docs/01_coil_pinout.md` and fill:
- a pin numbering sketch (P1..P5)
- resistance for all 10 pin pairs

Expected outcome:
- two low-resistance pairs (windings)
- one remaining pin (shield/drain)

### Step 2: Create the first EasyEDA sheet
EasyEDA sheet order:
1. Coil connector + RX protection + shield handling + test points
2. RX gain stage + bias + anti-alias filter
3. TX driver + current sense + protection
4. Power (battery input, reverse protect, regulators, analog filtering)
5. Debug/UI (SWD, UART, audio, buttons, optional OLED)

### Step 3: Firmware bring-up in phases
1. Generate stable single-frequency TX (low power)
2. Timer-triggered ADC sampling + DMA buffer
3. Single-frequency lock-in I/Q works
4. Add hopping scheduler (2 freqs, then 4)
5. Add baseline subtraction + ground balance
6. Add audio engine (threshold + target tones)

---

## Suggested V1 Defaults (tunable later)

### Frequency set (example)
- 5 kHz, 10 kHz, 15 kHz, 20 kHz

### Hopping schedule (example)
- 5 ms per frequency (20 ms total cycle, 50 Hz update rate)

### Safety rules
- Start TX at minimum amplitude
- Verify RX never clips at worst-case coupling
- Implement gain steps + clip detection early

---

## Build and Test Philosophy

OpenDD-MF is built like a product:
- every hardware revision exports PDFs + Gerbers + BOM
- every firmware revision can be tested with a repeatable protocol
- tuning changes are logged and explainable

---

## Licensing

This repository uses separate licenses for different parts of the project:

- **Firmware:** MIT (main GitHub license selection)
- **Hardware design files:** CERN-OHL-S v2 (planned)
- **Documentation:** CC BY 4.0 (planned)

See `LICENSES/` if present, or update these once the repo is initialized.

---

## Contributing

Contributions are welcome:
- hardware review (layout, grounding, protection)
- DSP improvements (lock-in filters, drift handling)
- coil characterization data (inductance/resistance + performance notes)
- documentation fixes (make the manual buildable by normal humans)

Use issues for:
- bug reports
- design proposals
- test results and measurement dumps

---

## Roadmap (V2+)

- simultaneous multi-tone TX mode
- target ID feature extraction across frequencies
- “profiles” (Park/Field/Beach style weighting)
- noise cancel scanning + interference metrics
- SD logging of I/Q features for offline tuning
- custom PCB to replace Nucleo for final builds

---

## Credits
Built by the OpenDD-MF community.
Initial direction and requirements defined by Ronin and Byte.
