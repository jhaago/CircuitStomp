# CircuitStomp Power Management

This document records the current planned power architecture for CircuitStomp Prototype 1 and the design constraints that should carry forward into later hardware revisions.

## Goals

The power system should:

- support normal battery-powered wireless stage use;
- allow charging while the controller is switched off;
- allow operation while external USB power is connected;
- keep the high-current LED subsystem off the XIAO regulator;
- provide enough headroom for the display, BLE, encoders, stomp switches, expression input and future status features;
- expose battery state to firmware;
- fail safely if LED firmware or wiring behaves unexpectedly;
- keep Prototype 1 modular enough that the final custom PCB can consolidate the same architecture later.

## Planned architecture

```text
                    USB-C CHARGE
                         |
                         v
                 +----------------+
                 | BQ24074 charger|
                 | + power sharing|
                 +-------+--------+
                         |
                  1S LiPo battery
                    3.7 V nominal
                         |
                    LOAD output
                         |
                  MASTER SWITCH
                         |
                         v
                5 V / ~2 A regulator
                         |
          +--------------+---------------+
          |              |               |
          v              v               v
     LED rings        XIAO 5 V        other loads
     9 x 12 LEDs          |
                          +--> onboard 3.3 V
                               |
                               +-- TFT
                               +-- MCP23017
                               +-- logic
```

Audio never passes through this power system or CircuitStomp itself. CircuitStomp remains a controller-only device.

## Charger and power-path controller

Prototype 1 should use an Adafruit BQ24074-based USB-C charger/power-path board or electrically equivalent implementation.

Reasons for this choice:

- single-cell LiPo charging;
- external USB power can run the load while also charging the battery;
- charging can continue while CircuitStomp's master power switch is off;
- selectable charge current provides a practical path for a larger stage-use battery;
- the architecture is suitable to reproduce later on a custom PCB.

The XIAO nRF52840 Plus onboard LiPo charger should not be treated as the primary CircuitStomp charger because its charge rate is not appropriate for the intended 5-6 Ah class battery.

The exact charge current must be set only after the selected battery's manufacturer-rated charging current is confirmed. A 1.5 A setting is an initial target, not a blanket requirement.

## Battery

Current target:

- protected 1S LiPo;
- approximately 5,000-6,000 mAh;
- 3.7 V nominal;
- suitable discharge capability for the regulated system rail;
- charge-current rating compatible with the selected BQ24074 configuration;
- physical size to be selected after the final internal enclosure volume is known.

A roughly 6,000 mAh pack is intended to provide comfortable rehearsal/gig runtime rather than merely proving battery operation.

The exact runtime target should be validated from measured Prototype 1 current consumption rather than estimated solely from component datasheets.

If the chosen battery exposes a thermistor, the charger should use it for temperature-aware charging where practical.

## Master power switch

CircuitStomp should use a real latching master power switch rather than relying on firmware sleep as the primary off state.

The switch should sit after the charger's load/power-path output and before the regulated system rail.

Desired behaviour:

```text
POWER OFF + USB-C connected
-> battery can charge
-> controller electronics remain off

POWER ON + USB-C connected
-> controller operates
-> battery may charge concurrently

POWER ON + no USB-C
-> controller runs from battery
```

The selected switch should be rated comfortably above expected DC system current. A rating of at least 3 A DC is preferred for design margin.

A recessed rocker or slide switch on the rear panel is currently preferred so it is unlikely to be knocked accidentally on stage.

## Regulated system rail

Prototype 1 should use a dedicated 5 V regulator capable of approximately 2 A continuous output, with suitable thermal and over-current protection.

A current candidate is the Pololu S13V20F5 or an equivalent 5 V step-up/step-down regulator.

The 2 A figure is design headroom, not an intended continuous operating current.

The high-current lighting subsystem must not be powered from the XIAO's onboard 3.3 V regulator.

## XIAO and logic power

The XIAO nRF52840 Plus should be powered from the regulated system rail, with its onboard 3.3 V regulator supplying the low-current logic domain where appropriate.

Likely 3.3 V-domain devices include:

- MCU logic;
- I/O expander;
- expression-input conditioning where appropriate;
- other low-current control electronics.

The exact Waveshare 2.4-inch TFT power arrangement should be confirmed against its module requirements during schematic design. Its SPI logic must remain electrically compatible with the XIAO's 3.3 V signalling.

## LED subsystem

The current physical design uses nine 37 mm addressable LED rings:

- six around rotary encoders;
- three around soft-touch stomp switches;
- 12 addressable RGB LEDs per ring assumed for current budgeting;
- 108 addressable LEDs total.

The encoder rings display continuous parameter/value information. The stomp rings primarily display solid state/colour information.

The design must not allow unrestricted full-brightness white operation across all LEDs as a normal operating mode.

Firmware should impose a global brightness/current budget. An initial target is approximately 500-600 mA maximum for the complete lighting system during normal operation, to be refined by measurement.

The LED rail should include:

- dedicated 5 V supply from the main regulated rail;
- resettable fuse/polyfuse, initially targeted around 1-1.5 A;
- appropriate bulk capacitance near the ring supply distribution;
- local decoupling where required;
- event-driven updates rather than unnecessary continuous refresh.

## LED data level shifting

The nRF52840 uses 3.3 V GPIO while the addressable LED rings are expected to operate from 5 V.

Prototype 1 should use a proper 3.3 V to 5 V logic-level translator for the LED data signal, such as a 74AHCT125 or equivalent.

The first LED data line should also include an appropriate series resistor, initially around 330-470 ohms.

The LED rings may be daisy chained so the complete lighting system can be driven from one translated data output if timing and wiring integrity remain reliable.

## Battery monitoring

CircuitStomp should expose useful battery state on its display rather than relying only on raw voltage thresholds.

A MAX17048-based 1S LiPo fuel gauge is the current preferred approach.

It can share the I2C bus with other low-speed peripherals where addressing and electrical loading allow.

Useful firmware-visible states include:

- approximate state of charge;
- low-battery warning;
- critically low battery state;
- external-power/charging indication where available.

## USB-C strategy for Prototype 1

Prototype 1 should keep charging/power and USB MIDI/programming as separate physical USB-C connections if necessary to avoid fragile rewiring of the XIAO USB data lines.

Current Prototype 1 concept:

```text
USB-C #1 - CHARGE / POWER
BQ24074 power-path board

USB-C #2 - MIDI / PROGRAMMING
XIAO nRF52840 Plus
```

Both may be exposed on the rear panel during development.

For the later custom PCB, the preferred direction is one consolidated USB-C connector:

```text
VBUS   -> CircuitStomp charger / power path
D+ / D- -> nRF52840 USB data
```

That later consolidation should preserve USB MIDI/programming while also supporting system power and battery charging.

## Protection and validation requirements

Before the power system is considered stage-ready, Prototype 1 should verify:

- measured idle current;
- measured typical LED/display operating current;
- worst-case permitted firmware current;
- battery runtime under representative use;
- regulator temperature during battery and USB-powered operation;
- charger temperature at the selected charge current;
- behaviour when USB power is connected and removed while running;
- behaviour when master power is switched on/off while charging;
- low-battery behaviour;
- LED over-current protection;
- battery protection operation;
- no brownouts during BLE activity, screen updates or rapid LED changes.

## Current provisional power BOM

| Function | Current planned part / requirement |
| --- | --- |
| Battery | Protected 1S LiPo, approximately 5,000-6,000 mAh |
| Charger / power path | Adafruit BQ24074 USB-C board or equivalent |
| System regulator | 5 V step-up/step-down, approximately 2 A; Pololu S13V20F5 candidate |
| Master switch | Latching SPST, preferably recessed, at least 3 A DC rating |
| Battery gauge | MAX17048 or equivalent 1S LiPo fuel gauge |
| LED data level shifter | 74AHCT125 or equivalent |
| LED protection | Approximately 1-1.5 A resettable fuse plus bulk capacitance |
| Controller | Seeed Studio XIAO nRF52840 Plus |
| Lighting | Nine 12-pixel 5 V addressable LED rings, subject to final part selection |

These selections are the current engineering baseline for Prototype 1, not an instruction to purchase every part before schematic, battery and mechanical checks are complete.
