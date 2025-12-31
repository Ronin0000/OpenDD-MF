# hardware/easyeda/README.md

# EasyEDA Hardware

This project uses EasyEDA for schematics and PCB.

## Versioning rules
- Every hardware change increments a version: V0.1, V0.2, V0.3...
- Every version exports:
  - schematic PDF
  - Gerbers
  - BOM export
  - Pick and place (if applicable)

Export locations in this repo:
- hardware/easyeda/exports/schematics_pdf/
- hardware/easyeda/exports/gerbers/
- hardware/easyeda/exports/bom/
- hardware/easyeda/exports/pick_and_place/

## Sheet plan (schematic)
SHEET 1: Coil connector + shield handling + RX input protection + bias + test points  
SHEET 2: RX gain stage + gain steps + anti-alias filter + ADC interface  
SHEET 3: TX interface (V0.1 external) and TX driver (V0.2 on-board)  
SHEET 4: Power (input protection, regulators, analog filtering)  
SHEET 5: Debug + UI (SWD, UART, audio, optional OLED)

## V0.1 scope
- Coil connector (5-pin)
- RX input protection
- Bias generator + buffer
- Simple RX gain stage footprinting + test points
- Header interface to NUCLEO-H723ZG (signals + power)

## V0.2 scope
- TX driver + current sense + protection
- TX enable / safety kill option

