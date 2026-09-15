# CircuitStomp Prototype 1 BOM

Date: 2026-09-15
Status: Working Prototype 1 bill of materials

This BOM records the current CircuitStomp Prototype 1 hardware baseline. Items are marked **Selected**, **Candidate**, or **TBD** so that parts can be ordered progressively without pretending unresolved mechanical or electrical choices are final.

## Status definitions

- **Selected** — architecture and part choice are sufficiently defined for Prototype 1.
- **Candidate** — preferred part or family is identified, but one or more checks remain before ordering.
- **TBD** — function is required, but the exact part is intentionally not yet selected.

## Main electronics

| Qty | Function | Status | Current part / requirement | Notes / dependency |
| ---: | --- | --- | --- | --- |
| 1 | Main controller | **Selected** | Seeed Studio XIAO nRF52840 Plus | BLE-capable nRF52840 controller with USB-C. Direct GPIO should be reserved primarily for encoder A/B signals and other timing-sensitive functions. |
| 1 | Display | **Selected** | Waveshare 2.4inch LCD Module, 240×320, ILI9341, 4-wire SPI, SKU 18366 | Module is 70.5 × 43.3 mm and fits the planned ~80 × 50 mm lower-left display zone. Intended landscape orientation in firmware. |
| 1 | Digital I/O expansion | **Candidate** | MCP23017 or equivalent I2C 16-bit I/O expander | Intended for lower-speed digital inputs such as encoder push switches and stomp switches. Exact need should be confirmed after final pin allocation. |
| 1 | Battery fuel gauge | **Selected** | MAX17048-based 1S LiPo fuel gauge | I2C battery state-of-charge reporting for the local display and low-battery warnings. |
| 1 | LED data level shifter | **Selected** | 74AHCT125 or equivalent 3.3 V → 5 V logic buffer | Used between nRF52840 GPIO and 5 V addressable LED data. Only one channel may be needed if all LED rings are daisy chained. |

## User controls

| Qty | Function | Status | Current part / requirement | Notes / dependency |
| ---: | --- | --- | --- | --- |
| 5 | Contextual pedal parameter encoder | **Candidate** | Bourns PEC11L-family endless incremental encoder with integrated momentary push switch; ~15 mm shaft target | Exact SKU must be selected together with the final knob. Low-profile body preferred. |
| 1 | Master output encoder | **Candidate** | Same Bourns PEC11L-family part as parameter encoders | Dedicated CircuitPedal master output control. Should mechanically match the five contextual controls. |
| 6 | Encoder knob | **TBD** | Low-profile / low-skirt knob with deep bore | This is the next critical mechanical selection. Knob geometry determines the final encoder shaft choice and top-panel stack height. |
| 3 | Stomp footswitch | **Candidate** | M12 SPST normally-open momentary soft-touch guitar-pedal footswitch | Must be genuinely comfortable underfoot. Panel-mounted directly through aluminium rather than into printed structure. Exact vendor/part still to be selected. |
| 1 | Expression input jack | **TBD** | 1/4-inch TRS panel-mount jack | Must support an external passive expression pedal. Input conditioning and polarity handling are still to be designed. |

## Control illumination

| Qty | Function | Status | Current part / requirement | Notes / dependency |
| ---: | --- | --- | --- | --- |
| 6 | Encoder LED ring | **Candidate** | 37 mm-class 12-pixel addressable RGB LED ring, 5 V | Displays parameter position/value. Exact PCB/module still to be selected. |
| 3 | Stomp LED ring | **Candidate** | Same 37 mm-class 12-pixel addressable RGB LED ring | Displays stomp state as a complete solid ring rather than a numerical level. Same visual language as encoder lighting. |
| 9 | Ring diffuser / light pipe | **TBD** | Custom translucent 3D-printed diffuser | Intended to hide discrete LEDs and create a continuous premium ring appearance above the aluminium panel. Geometry depends on exact ring PCB and panel stack-up. |
| 1 | LED series resistor | **Selected** | 330–470 ohm | Series resistor on the first addressable LED data line. Final value can be selected during wiring validation. |
| 1 | LED rail bulk capacitor | **Candidate** | Large electrolytic capacitor on the 5 V LED rail | Exact value to be chosen during schematic design based on wiring length and measured transients. |
| 1 | LED rail resettable fuse | **Candidate** | ~1–1.5 A polyfuse | Hardware backstop for the lighting subsystem. Firmware will also enforce a global brightness/current budget. |

## Power system

| Qty | Function | Status | Current part / requirement | Notes / dependency |
| ---: | --- | --- | --- | --- |
| 1 | Battery | **Candidate** | Protected 1S LiPo, approximately 5,000–6,000 mAh | Exact pack dimensions depend on final internal enclosure volume. Charge/discharge ratings must be checked before purchase. |
| 1 | Charger / power-path | **Selected** | Adafruit BQ24074 USB-C charger / power-path board or electrically equivalent implementation | Allows load sharing and charging while CircuitStomp master power is off. Charge current must be configured to suit the selected battery. |
| 1 | 5 V system regulator | **Candidate** | Pololu S13V20F5 5 V step-up/step-down regulator | 2 A-class design target. Confirm thermal performance with the final LED/display load. |
| 1 | Master power switch | **TBD** | Latching SPST recessed slide or rocker switch, >=3 A DC preferred | Mounted on rear panel after charger LOAD output and before the regulated system rail, allowing battery charging while the controller is switched off. |
| 1 | Prototype USB-C charge/power connector | **Selected for Prototype 1 architecture** | USB-C on BQ24074 board | Dedicated power/charge port for Prototype 1 if separate from XIAO USB data. |
| 1 | Prototype USB-C MIDI/programming connector | **Selected for Prototype 1 architecture** | XIAO onboard USB-C | Used for firmware, debugging and USB MIDI fallback. A later custom PCB should aim to consolidate data and charging into one USB-C port. |

## Mechanical structure

| Qty | Function | Status | Current part / requirement | Notes / dependency |
| ---: | --- | --- | --- | --- |
| 1 | Structural shell | **Candidate** | Bent aluminium saddle shell, approximately 2–2.5 mm sheet | Top and rear in one bent aluminium structure; open sides closed with printed endcaps. Exact width/depth/bend radius remain to be dimensioned from real parts. |
| 2 | Side/end panels | **TBD** | 3D-printed polymer endcaps | Also provide useful 2.4 GHz RF windows. XIAO antenna should be located near one plastic end rather than buried centrally beneath aluminium. |
| 1 set | Internal mounts | **TBD** | 3D-printed brackets/rails/cradles | Expected for XIAO, display, battery, LED rings, power electronics and wiring support. Heat-set threaded inserts preferred where repeated service access is expected. |
| 1 | Display cradle | **TBD** | 3D-printed internal display bracket | Supports the Waveshare display behind the aluminium top. Optional clear acrylic/polycarbonate window can be added later. |
| 1 | Battery cradle | **TBD** | 3D-printed battery restraint | Must prevent battery movement under transport/stomping without crushing or puncturing the pouch cell. |
| 1 set | Base / feet | **TBD** | Non-slip rubber feet and removable/serviceable bottom solution | Must prevent movement when stomp switches are used. |

## Wiring and interconnect

| Qty | Function | Status | Current part / requirement | Notes / dependency |
| ---: | --- | --- | --- | --- |
| 1 set | Internal low-current connectors | **TBD** | JST-PH / JST-GH / equivalent locking connectors | Exact family to be chosen once board arrangement is known. Avoid unnecessary permanent point-to-point wiring where modules may need removal. |
| 1 set | Power wiring | **TBD** | Appropriately rated stranded wire and connectors | Gauge must be selected for LED/system current and connector length. |
| 1 | Main carrier / distribution board | **TBD** | Prototype carrier PCB, perfboard, or structured wiring harness | Central PCB is likely cleaner, but should follow confirmed pin allocation and physical layout rather than being designed prematurely. |
| up to 6 | Repeated encoder sub-board | **TBD** | Optional small encoder/LED interface PCB | Only justified if it materially simplifies final wiring and mechanical assembly. |

## Firmware-facing hardware assignments still to define

These do not require new components immediately but must be resolved before a schematic/carrier PCB is frozen:

- exact XIAO pin allocation;
- exact MCP23017 usage and I2C addressing;
- SPI pins and display control lines;
- addressable LED data topology;
- encoder push-button semantics;
- three stomp-switch semantics;
- expression input analogue circuit and polarity support;
- external power / charge-state sensing;
- battery critical shutdown behaviour;
- final MIDI CC / relative encoder mapping.

## Safe to order now

These items are sufficiently independent of unresolved mechanical dimensions to purchase for bench work:

1. Seeed Studio XIAO nRF52840 Plus — 1
2. Waveshare 2.4inch LCD Module, 240×320 ILI9341 — 1
3. Adafruit BQ24074 USB-C charger / power-path board — 1
4. MAX17048 fuel-gauge breakout — 1
5. 74AHCT125 logic buffer / suitable breakout or IC — 1
6. MCP23017 breakout or IC — 1 for prototype experimentation

## Hold before ordering

Do not lock these until their dependency is resolved:

- **knobs** — select exact physical style first;
- **encoders** — select exact PEC11L SKU only after knob bore/depth and shaft requirement are known;
- **battery** — wait for internal envelope and acceptable charge/discharge specification;
- **soft-touch footswitches** — select exact part after verifying desired actuator diameter/height and preferably physically testing one;
- **LED rings** — confirm exact PCB dimensions and diffuser geometry before buying all nine; buying one for mechanical prototyping first is sensible;
- **5 V regulator** — the Pololu S13V20F5 remains the preferred candidate, but verify measured current and thermal margin once one LED ring + display are on the bench;
- **master switch, TRS jack, connectors and wiring** — choose after the rear-panel and carrier-board layout is dimensioned.

## Prototype 1 ordering principle

Prototype 1 should validate the interaction model and power/control architecture without prematurely turning every provisional choice into a production commitment. Where a mechanical item can affect several downstream dimensions, buy one sample first, measure it, and only then order the full quantity.
