# CircuitStomp Initial Repository Design

Date: 2026-09-15
Status: Approved design baseline for repository setup

## 1. Purpose

CircuitStomp is a wireless stage foot controller designed primarily as a companion to CircuitPedal, while remaining useful as a standards-based MIDI controller where practical.

The controller exists to make CircuitPedal practical for live performance without requiring the player to reach for an iPad or MacBook for routine actions. It should provide tactile foot controls for switching and stage functions, rotary controls for real-time pedal adjustment, and a small local display for essential feedback.

CircuitStomp is a separate repository from CircuitPedal because the hardware, firmware, enclosure, power system and MIDI transport are distinct engineering concerns. CircuitPedal remains the host application and source of pedal/board state; CircuitStomp is the physical controller.

## 2. Core philosophy

### 2.1 Stage reliability before novelty

The controller is live-performance equipment. Predictability, reconnect behaviour, low interaction latency, physical robustness, clear status indication and graceful fallback matter more than adding features quickly.

### 2.2 Bluetooth carries control, not instrument audio

Guitar/bass audio remains on the wired audio-interface path. Bluetooth is used only for controller data. This keeps wireless transport latency and radio instability out of the audio signal path.

### 2.3 BLE MIDI is the primary wireless transport

CircuitStomp should use Bluetooth Low Energy MIDI for normal wireless operation rather than a proprietary Bluetooth-only protocol.

This keeps the controller broadly compatible with MIDI-capable hosts and avoids tying the basic control transport permanently to one CircuitPedal implementation.

### 2.4 USB MIDI is the wired fallback

The controller should also support USB MIDI through its USB-C connection where practical.

USB should serve as:

- a reliable fallback if Bluetooth is unsuitable or fails at a venue;
- a firmware/programming connection;
- a charging/power path;
- a useful development and debugging path.

### 2.5 CircuitPedal is authoritative for rig state

CircuitStomp should not become the source of truth for pedalboard state.

The host application owns:

- which pedals are loaded;
- selected pedal;
- bypass state;
- parameter values;
- board/session state;
- scenes/presets;
- master output state.

CircuitStomp sends user intent and displays feedback from the host. This makes state changes from either the software UI or the physical controller converge on the same authoritative state.

### 2.6 Standard MIDI first, CircuitPedal extensions second

Normal control should use conventional MIDI messages where they fit. CircuitPedal-specific behaviour may later use a narrow extension protocol, likely SysEx or another MIDI-compatible mechanism, for richer metadata such as selected pedal identity, parameter labels, scene names, tuner data or display text.

The basic controller should remain useful even without those extensions.

### 2.7 Physical controls should work without constant screen attention

The controller must remain usable by feel on stage. The local display is for confirmation and context, not a requirement for every basic action.

Dedicated controls should stay dedicated where that materially improves live usability.

### 2.8 Hardware and firmware should remain modular

Input scanning, MIDI transport, display rendering, connection state and CircuitPedal-specific extensions should be separate firmware responsibilities so that individual subsystems can be changed or tested without rewriting the entire controller.

The same principle applies to hardware: prototype wiring, carrier PCB, enclosure and final production-style PCB are separate stages.

### 2.9 Design for progression, not premature production hardware

Prototype 1 should validate the real interaction model with inexpensive modular hardware. A custom PCB and final enclosure come only after the controls, wireless behaviour and stage workflow have been physically tested.

## 3. Planned Prototype 1 physical configuration

Prototype 1 is intended to be representative enough to test real CircuitPedal stage use, not merely prove that a BLE packet can be sent.

### 3.1 Main controller board

Current planned controller:

- Seeed Studio XIAO nRF52840 Plus
- nRF52840-based BLE-capable microcontroller
- USB-C connector
- onboard battery/charging capability to be used if it proves suitable during prototype validation

The board choice is provisional until the actual pinout, library/toolchain support and electrical requirements are confirmed against the final Prototype 1 wiring design.

### 3.2 Rotary controls

Six endless rotary encoders are planned.

Five are contextual pedal-parameter controls:

- Parameter 1
- Parameter 2
- Parameter 3
- Parameter 4
- Parameter 5

Their meaning changes with the selected pedal. Examples:

- a three-parameter pedal may use only P1-P3;
- Woolly Mammoth can map Wool, Pinch, EQ and Output across four controls;
- a five-control Fuzz Factory-style pedal can use all five;
- future effects with more than five parameters may use pages rather than requiring more permanent knobs.

The sixth encoder is dedicated to CircuitPedal master output. It is intentionally separate from a selected pedal's own Volume/Level/Output parameter.

All six should be endless encoders rather than absolute-position potentiometers so control values do not jump when the selected pedal or preset changes.

Pushable encoders are preferred if suitable components are selected, but the push actions are not yet assigned and should not be treated as required behaviour until stage workflows justify them.

### 3.3 Foot controls

Three momentary stomp switches are planned for Prototype 1.

Their final semantic mapping is intentionally not fixed yet. Candidate functions include:

- pedal bypass/engage;
- previous/next pedal or effect slot;
- scene/preset switching;
- tuner;
- tap tempo;
- modifier/secondary actions.

Prototype 1 should favour simple, observable functions before layering short-press/long-press combinations.

### 3.4 Display

A small, simple local display is planned.

The initial preference is a small I2C monochrome OLED because Prototype 1 needs low-complexity status feedback rather than a graphical user interface.

Useful initial information includes:

- Bluetooth/USB connection state;
- selected pedal name;
- current parameter labels/values where space permits;
- master output change feedback;
- stomp-state confirmation;
- battery status later if practical.

A colour display is not required for Prototype 1.

### 3.5 I/O expansion

An MCP23017 or similar I2C I/O expander is currently planned for slower digital inputs such as encoder push buttons and stomp switches.

The preferred architecture is:

- direct MCU GPIO for time-sensitive rotary A/B signals;
- I2C expander for lower-speed push/stomp inputs;
- the display sharing the I2C bus if electrically suitable.

This avoids consuming the entire microcontroller pin budget and leaves room for future expansion.

### 3.6 Power

Prototype 1 should support rechargeable battery operation.

Current intent:

- single-cell LiPo battery;
- USB-C charging/power through the controller board where safe and sufficient;
- battery operation for normal wireless stage use;
- USB operation for development and fallback.

Battery capacity, runtime target, power switch implementation, low-voltage behaviour and charging safety remain to be determined during hardware design.

### 3.7 Enclosure

Prototype 1 may use a 3D-printed or otherwise easily modified enclosure.

The final enclosure must account for 2.4 GHz radio performance. The antenna should not be buried inside a fully shielding metal enclosure without an RF-aware solution such as placement near a non-metallic section, RF window or suitable antenna arrangement.

Mechanical priorities include:

- stomp-switch spacing suitable for shoes;
- encoder protection from accidental foot impact;
- clear separation of the dedicated master control;
- accessible USB-C;
- secure battery mounting;
- serviceability during development.

## 4. Proposed physical control concept

The current working arrangement is:

```text
                 [ simple display ]

 P1     P2     P3     P4     P5          MASTER
  O      O      O      O      O             O

        [ SW1 ]       [ SW2 ]       [ SW3 ]
```

The exact dimensions and spacing should be decided from real component dimensions and a physical mock-up rather than from a screen drawing alone.

The master encoder should be visually or spatially differentiated from the five contextual parameter encoders.

## 5. Host/controller relationship

Conceptually:

```text
Guitar/Bass
    |
    v
USB Audio Interface ---> iPad or Mac ---> Audio Interface Output
                              |
                         CircuitPedal
                              ^
                              |
                    BLE MIDI / USB MIDI
                              |
                         CircuitStomp
```

Audio never routes through CircuitStomp.

A typical interaction should be:

1. user turns an encoder or presses a stomp switch;
2. CircuitStomp emits the appropriate MIDI event;
3. CircuitPedal applies the requested change to authoritative board state;
4. CircuitPedal optionally returns state/metadata to CircuitStomp;
5. CircuitStomp updates LEDs/display without maintaining an independent conflicting copy of the rig.

## 6. Firmware architecture direction

Firmware should be organised around small responsibilities rather than one monolithic event loop.

Likely modules:

- input scanning and debounce;
- rotary encoder decoding;
- logical control mapping;
- BLE MIDI transport;
- USB MIDI transport;
- connection/pairing state;
- display/status rendering;
- host feedback/state cache;
- power/battery monitoring;
- optional CircuitPedal extension protocol.

Input handling should remain responsive even while the display is updating or Bluetooth reconnects.

The firmware should avoid hidden assumptions that a particular physical switch always corresponds to a particular pedal model.

## 7. MIDI/control protocol direction

Prototype 1 should first validate conventional MIDI messages.

Potential mapping families include:

- Control Change for rotary parameters and selected global controls;
- Program Change or defined CC messages for scene/preset selection;
- note/CC style momentary messages for stomp actions;
- future SysEx for CircuitPedal-specific metadata and bidirectional display/state information.

Exact channel numbers, CC assignments, relative encoder format and extension-message definitions should be documented in a dedicated protocol document before firmware and CircuitPedal depend on them.

Relative encoder messages are preferred over pretending an endless encoder has an absolute physical position.

## 8. Repository structure

The intended initial structure is:

```text
CircuitStomp/
|-- README.md
|-- docs/
|   |-- philosophy.md
|   |-- hardware_plan.md
|   |-- control_layout.md
|   |-- midi_protocol.md
|   `-- roadmap.md
|-- firmware/
|   `-- README.md
|-- hardware/
|   |-- README.md
|   |-- pcb/
|   |-- enclosure/
|   `-- bom/
`-- prototypes/
    `-- prototype-1/
        `-- README.md
```

The `docs/superpowers/specs/` directory contains design-process records and is not part of the product-facing documentation hierarchy.

## 9. Prototype roadmap

### Phase 1 - repository and design baseline

Create the project documentation, hardware assumptions, protocol placeholder and Prototype 1 definition before firmware implementation begins.

### Phase 2 - bench Prototype 1

Assemble enough hardware to validate:

- six encoder rotation inputs;
- three stomp switches;
- basic display output;
- BLE MIDI connection;
- one or more live CircuitPedal mappings;
- USB MIDI feasibility/fallback.

The first success criterion is not enclosure polish. It is reliable physical control of CircuitPedal.

### Phase 3 - bidirectional controller behaviour

Add host-to-controller feedback where useful:

- selected pedal;
- bypass/engage state;
- parameter metadata;
- parameter value feedback;
- connection status;
- scene information.

### Phase 4 - stage hardening

Test and improve:

- pairing and automatic reconnection;
- behaviour after host sleep/app restart;
- RF range and interference tolerance;
- switch debounce;
- encoder responsiveness;
- accidental double actions;
- battery endurance;
- low-battery behaviour;
- USB fallback;
- readability in dark/bright stage environments.

### Phase 5 - refined hardware

Only after the control model is physically validated:

- custom carrier PCB or integrated PCB;
- final connector strategy;
- refined power design;
- robust enclosure;
- RF-conscious mechanical design;
- production-quality wiring/connectors;
- final BOM.

## 10. Future capabilities, not Prototype 1 commitments

The architecture should leave room for, but not prematurely require:

- tuner mode and tuner display;
- scene/preset browsing;
- tap tempo;
- expression-pedal input;
- parameter pages beyond the five primary contextual knobs;
- per-switch LEDs or RGB indicators;
- richer host-driven display layouts;
- battery percentage reporting;
- user configuration/mapping;
- wired operation as a full alternative to BLE;
- support for MIDI software other than CircuitPedal.

## 11. Explicit non-goals for the first prototype

Prototype 1 is not intended to be:

- a wireless audio interface;
- a finished commercial pedalboard;
- a touchscreen controller;
- a colour-graphics UI project;
- a custom PCB-first build;
- a replacement for CircuitPedal's authoritative board/session model;
- an attempt to implement every future stomp function at once.

## 12. Success criteria for the initial project direction

The repository and Prototype 1 direction are successful if they allow the project to demonstrate that:

1. a player can reliably connect CircuitStomp to CircuitPedal wirelessly;
2. six physical encoders can control five contextual pedal parameters plus master output without value jumps;
3. three stomp switches can trigger dependable live actions;
4. the small display provides enough confirmation to use the controller without constantly looking at the host screen;
5. wired USB MIDI can serve as a practical fallback if adopted after prototype validation;
6. the controller architecture remains standards-friendly while allowing richer CircuitPedal-specific feedback later;
7. the design can progress to a custom PCB and rugged enclosure without discarding the validated firmware/control model.
