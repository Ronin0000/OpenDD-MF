# Coil Interface Standard (Passive DD, 5-pin)

This project uses a logical 5-pin coil interface so any passive DD coil can be adapted.

## Connector signals (schematic standard)
J1-1: TX+
J1-2: TX-
J1-3: RX+
J1-4: RX-
J1-5: SHIELD_DRAIN

## Builder wiring note
Your physical coil connector pin numbering may differ.
Map your coil wires to the above signals using continuity and resistance checks.

## Shield/drain rule
Connect SHIELD_DRAIN at one point only (control box end).
Do not connect shield at both ends.

## Coil profile concept
Coils differ in resistance, inductance, coupling, and balance.
Firmware supports multiple coils using a "Coil Profile" (see firmware docs).

