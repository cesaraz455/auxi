# CLAUDE.md

Guidance for Claude Code when working in this repository. This file is the **complete design specification** for the project. No external document is needed.

---

## 1. Project overview

**auxi** is a small soccer robot, a first-semester university project. This repo holds the KiCad design for its **single custom PCB**.

- **Drive:** 2 driven wheels (yellow TT gear motors) plus 1 rear caster wheel.
- **Chassis:** 3D-printed, about 150 × 150 mm. The front of the chassis has a **concave area that guides the ball**. It is part of the chassis itself, not a separate part.
- **Two-level architecture:**
  - **Bottom level (chassis):** batteries, motors, caster and wiring.
  - **Top level:** a **single PCB** on M3 standoffs, above the batteries and motors.
- **The PCB carries:**
  - The plug-in modules: ESP32-S3-DevKitC-1, ESP32-CAM and the TB6612FNG breakout.
  - The MP1584 regulator.
  - The passive components.
  - **All connectors.**
- **Everything else is off-board** and connects to the PCB through headers or connectors:
  - The 3 VL53L0X sensors.
  - The 3 proximity LEDs and the power LED.
  - The switch.
  - The motors.
  - The battery.
  - The camera.
- **Main ESP32:** controls the robot.
  - Receives commands from an **Xbox controller over Bluetooth Low Energy**.
  - Reads the 3 sensors.
  - Drives the TB6612FNG and the LEDs.
- **ESP32-CAM AI Thinker:**
  - Streams live video over Wi-Fi to a phone.
  - Shows the distances it receives from the main ESP32 over UART.
  - Its OV2640 camera is fixed at the front of the chassis on a long flex cable.
- **Sensors:** 3, mounted in the concave front area: FRONT (0°), LEFT (+45°), RIGHT (−45°).
- **Proximity LEDs:** 3. Their behavior depends on the sensors and follows the university rules (§10).

---

## 2. How to work

- **Use the KiCad MCP tools** (`mcp__kicad__*`) to create, edit, verify and organize the real project files. Do not hand-edit `.kicad_sch` or `.kicad_pcb` S-expressions unless the MCP cannot do the job.
- Treat this as a real electronics engineering project. **Never invent** pinouts, dimensions, footprints or electrical specs. If a fact is not in this file or in verifiable component documentation, **stop and ask** (for example, ask for photos or caliper measurements of the purchased module).
- **§3 is the single source of truth.** If you find a contradiction, **do not resolve it silently**: explain it and propose a fix.
- **The user is a beginner.** Explain each decision in plain language, prefer simple, robust and easy-to-assemble solutions, and **do not add features that were not requested**.
- **Language:** the user writes in Spanish, so answer in Spanish. Repository documentation (`README.md` and this file) is in English.
- **KiCad lock files:** `~*.lck` files mean KiCad has the project open in the GUI. Warn the user before writing to files they may be editing.
- **Git:** commit only when the user asks.

---

## 3. Confirmed decisions (single source of truth)

| Topic | Decision |
|---|---|
| Number of PCBs | **Exactly one custom PCB.** Commercial modules do not count as extra PCBs. |
| PCB layers and parts | **1 copper layer (`B.Cu`, the bottom face) with a GND pour on the same layer.** `F.Cu` has no copper. **All components THT** (no SMD), mounted on the top face and **soldered from below**. No plated holes, no vias and no jumper wires. |
| PCB location | Top level, on 4 M3 standoffs, above the batteries and motors. |
| PCB size | **110 × 140 mm**, set by the user (it was 90 × 100 mm; the extra room is for the single-layer routing). Do not enlarge it without approval. The 110 mm side runs front to back and the 140 mm side across the robot (assumed; confirmed during the space check, §4.3). |
| On the PCB | ESP32-S3-DevKitC-1, ESP32-CAM, TB6612FNG module, MP1584 module, capacitors, resistors, diode and connectors. |
| Off-board (with a connector on the PCB) | 3 VL53L0X sensors, 3 proximity LEDs, power LED, switch, 2 motors, battery and OV2640 camera. |
| Chassis front | Concave area built into the chassis to guide the ball; not a separate part. |
| Battery | 2 × 18650 in series (2S): 6.0 V minimum, 7.4 V nominal, 8.4 V maximum. On the bottom level, under the PCB. |
| Battery connector | XT30 (polarized): BAT+ / BAT−. |
| Switch | Off-board, on the chassis floor. Sits between the battery and the regulator and cuts **the whole** system. |
| Power LED | Blue, off-board, next to the switch. Connected after the switch; no GPIO controls it. |
| Main ESP32 | **ESP32-S3-DevKitC-1 N16R8** (16 MB flash, 8 MB octal PSRAM, 44 pins in 2 rows of 22). It may be a clone: verify it against Espressif's documentation. |
| Controller | Xbox controller over Bluetooth Low Energy (the ESP32-S3 has no Classic Bluetooth). The exact model and library are chosen in firmware. |
| Motors | 2 × yellow TT gear motors. TB6612FNG VM comes after the switch; firmware limits PWM to 70 % maximum. |
| Driver | Commercial TB6612FNG breakout, plugged into the PCB through female headers. |
| 5 V regulator | Adjustable MP1584EN mini module, set to 5.0 V. |
| 3.3 V | From the 3V3 regulator on the main ESP32 board. |
| Sensors | 3 × VL53L0X (GY-530 module) at **3.3 V**, shared I2C bus, one independent XSHUT per sensor. |
| Proximity LEDs | Off-board. Colors and threshold follow the university rules: FRONT red, RIGHT green, LEFT yellow; on at ≤ 10 cm. |
| ESP32-CAM | AI Thinker, no microSD, **removable** (programmed by removing it and using the user's ESP32-CAM-MB base), no programming header. |
| Camera | OV2640 with a long 24-pin, 0.5 mm pitch flex, fixed at the front of the chassis. |
| ESP32 ↔ ESP32-CAM UART | Direct traces on the PCB; no connector. |
| Home-made PCB rules | Tracks ≥ 0.4 mm (0.3 mm allowed where a track passes between pads). Holes ≥ 0.3 mm. **Pad diameter = drill diameter + 0.8 mm** on every pad hole, so copper is left on the pad after hand drilling. See §14. |
| Do NOT include | Battery monitoring, programming header, MOSFET protection, any other unrequested feature. |

---

## 4. Mechanical architecture

> For the schematic stage, use this section only to choose footprints and connectors that fit the final assembly.

### 4.1 Chassis (based on the user's concept sketch)

Coordinates: origin at the center, X to the right, Y forward.

- Outer outline: about 150 × 150 mm.
- **Concave front area:** reaches Y ≈ +42 mm in the center and Y ≈ +73 mm at the corners.
- TT motors on the sides. In the sketch they occupy X ≈ ±52 to ±75. **The real TT length must be measured.**
- Rear caster at Y ≈ −55 mm.
- Sensors in the concave area:
  - FRONT in the center, facing +Y.
  - LEFT and RIGHT near the ends of the curve, pointing at +45° and −45°.
- OV2640 camera at the front, facing forward.

The sketch is a concept, not a final drawing. Confirm the measurements with the user.

### 4.2 Two levels (side view)

```
   ┌──────────────── single PCB (top level) ───────────────────┐
   │ ESP32-S3 · ESP32-CAM · TB6612 · MP1584 · connectors       │
   └───┬──────────────────────────────────────────────────┬────┘
       │ M3 standoff                          M3 standoff │
   ════╪══════ batteries · TT motors · wiring ════════════╪════
            chassis floor (switch and power LED underneath)
```

**Why two levels:** the chassis floor is small. The motors take the sides, the concave area takes the front and the caster takes the back. With two levels, the batteries and motors use the whole floor and the PCB uses the whole top area. It also has these benefits:

- The battery weight stays low, which makes the robot more stable.
- Cables run up short distances to their connectors.
- The USB ports, the ESP32-CAM and the connectors are reachable from above.

### 4.3 Mounting, height and space check

This check is mandatory before PCB design. It does not block the schematic.

- **Mounting holes:** the PCB has 4 M3 holes, one near each corner.
  - Each hole is 3.2 mm, with a keepout of about 6.5 mm diameter (no copper or parts), at least 4 mm from the edge.
  - The standoffs must rest on the chassis in areas free of batteries, motors and wheels.
- **Standoff height:** it must keep the PCB above the battery holder, the motors and the cables.
  - Leave at least 5 mm between the bottom-side solder joints and any metal part of the batteries (holder contacts).
  - If that cannot be guaranteed, recommend an insulating sheet under the PCB.
  - Use standard M3 lengths and document which one.
- **Wheels:** if the wheels are inside the 150 × 150 mm outline, the PCB must not hit them. Check the wheel height (50 mm diameter) against the PCB height.
- **Concave area:** the PCB must not block the ball entry or the camera view, unless the user approves it.
- **Cable path:** the enclosure and chassis must let the battery, motor, sensor, LED, switch and camera cables run up from the bottom level to their connectors.
- **Size: 110 × 140 mm, set by the user.**
  - The layout assumes the 110 mm side runs along the robot (front to back) and the 140 mm side across it. Confirm this with the user. The 140 mm side leaves only 5 mm per side inside the 150 mm chassis outline, so check the wheels and the standoff positions.
  - Check that all modules, connectors, M3 holes and antenna keepouts fit that outline, with room to route and to plug cables in by hand.
  - The ESP32-S3-DevKitC-1 is about 63–68 mm long. Its USB end is about 5 mm inside the bottom edge, so a plug reaches it from the side and overhangs the board edge. Keep the space around the USB end free of tall parts (the plug needs about 15 × 12 mm there).
  - If something does not fit, **do not enlarge the board**. Explain the problem and propose alternatives before continuing.
- Present a 2D check (top and side views) with real measurements and **wait for approval**.
- Evaluate the weight distribution: the drive wheels must keep traction. Report if relevant.

### 4.4 Reference dimensions (verify with calipers)

These are reference values for the physical layout. Clones vary: confirm them with the user before fixing positions.

| Component | Approx. dimensions (mm) | Note |
|---|---|---|
| ESP32-S3-DevKitC-1 | 62.74 × 25.40 official; user's clone ~68 × 25 | Official DXF: headers 1.27 mm from each long edge, so **rows 22.86 mm apart**; USB at the pin-22 end, antenna at the pin-1 end |
| ESP32-CAM AI Thinker | 26.9 × 39.8 (seller photo) | Rows **22.86 mm apart** (common value); pins at the microSD end, antenna and FPC at the other end (assumed) |
| TB6612FNG module | 20.5 × 20.4 (seller photo, red generic module) | Row spacing **assumed 17.78 mm (0.7")**; verify |
| MP1584EN mini module | 22.5 × 17.2 × 4 (seller photo, D-SUN type) | 8 through-holes, 2 per terminal (2.54 mm pitch), ~19.9 mm between sides (estimated) |
| 2×18650 battery holder | ~75–78 × ~40 × ~18–22 | Bottom level |
| TT motor (body) | 70 × 22 × 18; ~37 wide including both shafts | Bottom level, plus the wheel |
| VL53L0X GY-530 | ~25 × 11 | Off-board, in the concave area |
| JST-XH 5-pin vertical | ~14.9 × 5.75 × 7 | One per sensor |
| 2-pin male header, 2.54 mm | ~5.1 × 2.5 × 8.5 | LEDs |
| 2P screw terminal, 5.08 mm | ~10.2 × ~8 × ~10 | Motors and switch |
| PCB-mount XT30 | Use the Amass drawing | Check vertical vs. horizontal model |
| 470 µF electrolytic | ~Ø10 × 13–20 | Height depends on voltage and brand |
| M3 hole | Ø3.2 with ~Ø6.5 keepout | 4 corners |

Modules plugged into female headers sit about 1 cm above the PCB. Account for this in the enclosure height.

---

## 5. Power

### 5.1 Architecture

```
BAT 2S ─ XT30 ─ J_SWITCH ═(wires)═ external switch ═(wires)═ J_SWITCH ─ +VBAT_SW
                                                                        │
+VBAT_SW ─┬─ resistor ─ J_LED_POWER ─(wires)─ external blue LED ─ GND
          ├─ bulk capacitor ─ TB6612 VM (motors)
          └─ MP1584 ─ +5V ─┬─ Schottky diode ─ +5V_ESP ─ ESP32 5V pin
                           └─ ESP32-CAM 5V pin
ESP32 3V3 pin ─ +3V3 ─┬─ 3 × VL53L0X
                      ├─ TB6612 VCC (logic)
                      └─ I2C pull-ups (if needed)
```

**Rule:** nothing connects to the battery before the switch. With the switch off, the regulator, the motor driver and all LEDs are unpowered.

### 5.2 Motors and PWM

- TT motors are typically rated 3–6 V. **Verify this in the seller's datasheet.**
- VM is fed from `+VBAT_SW`. The firmware limits PWM to a maximum of 70 % (0.7 × 8.4 V ≈ 5.9 V average). Document this as a mandatory firmware rule.
- **Do not raise the battery voltage.**
- **Stall current:** get the real motor's stall current, or ask the user to measure it. Compare it with the TB6612FNG limits in Toshiba's datasheet. The usual figures are 1.2 A continuous and 3.2 A peak per channel; verify them. If the driver is not enough, report it before continuing.

### 5.3 Regulators

- **MP1584:**
  - Verify the pinout and dimensions of the real module; clones differ.
  - Compute the 5 V current budget from datasheets: ESP32-S3 with Bluetooth, ESP32-CAM with Wi-Fi and camera, sensors and LEDs.
  - Check the margin and temperature.
- **ESP32 3V3:** check in the official ESP32-S3-DevKitC-1 schematic that its regulator can supply the ESP32, the 3 sensors, the TB6612 logic and the 3 LEDs with enough margin.
- **Schottky diode** (SS14 / 1N5819 class, ≥1 A) between `+5V` and the ESP32 5V pin.
  - It keeps the computer's USB from powering the robot, and the robot from powering the USB, while programming.
  - Check that the resulting voltage (about 4.5–4.7 V) is enough for the board's regulator.
  - If it is not, propose setting the MP1584 slightly higher (for example 5.2 V) and check that the ESP32-CAM tolerates it (its typical range is 4.75–5.25 V).
  - **Implemented:** D1 = 1N5819 (DO-41). The ESP32 5V pin gets about 4.5–4.6 V, which is still enough for its 3.3 V regulator.

**Current budget (estimated from typical datasheet values):**

| Load | Rail | Estimated peak |
|---|---|---|
| ESP32-S3 with BLE | 3V3, from `+5V_ESP` | ~350 mA |
| 3 × VL53L0X | 3V3 | ~60 mA |
| 3 LEDs + TB6612 logic | 3V3 | ~15 mA |
| ESP32-CAM with Wi-Fi and video | `+5V` | ~500 mA |
| **Total on `+5V`** | | **~1.0 A** |

- The MP1584 has plenty of margin at ~1 A (the module is rated up to 3 A).
- From the battery, the electronics draw ~0.75 A at 7.4 V, plus the motors.

### 5.4 External switch and power LED

- **Location:** the switch is **on the chassis floor, off the PCB**, reachable from below without disassembling anything.
- **Electrical position:** it sits **between the battery and the regulator**. Per the rule in §5.1, it also cuts VM and all LEDs.
- **J_SWITCH on the PCB:** 2 terminals, `SW_BAT` (from the battery) and `SW_OUT` (to `+VBAT_SW`).
  - **All** of the robot's current flows through here, including motor start-up peaks.
  - Use a connector with a verified rating of ≥ 3 A. Recommended: a 2-pin, 5.08 mm pitch screw terminal, which needs no crimping.
  - **Do not use 2.54 mm headers with Dupont wires for this path.** Dupont jumpers handle about 1–2 A and their contact loosens.
- **Switch part:**
  - Continuous rating ≥ 3 A DC and ≥ 12 V DC, **verified in the datasheet**.
  - Estimate the system peak current and justify the choice.
  - Recommend a panel-mount switch: a slide switch if one exists with that rating, otherwise a mini rocker.
  - State the minimum wire gauge.
- **J_LED_POWER on the PCB:** 2-pin male header, 2.54 mm pitch (`LED+` / `LED−`), for the off-board blue LED next to the switch, connected with Dupont jumpers.
  - The current-limiting resistor is on the PCB. Size it for the whole 6.0–8.4 V range at about 2–5 mA.
  - **Implemented:** R1 = 1.5 kΩ, which gives 2.0 mA at 6.0 V and 3.6 mA at 8.4 V.
- **Silkscreen:** `SWITCH` / `ON-OFF` on J_SWITCH; `POWER LED` and polarity on J_LED_POWER.

### 5.5 Battery

- No MOSFET protection is added. Reverse-polarity protection comes from the polarized XT30 connector plus the `BAT+` / `BAT−` silkscreen.
- BMS and cell charging are out of scope for the PCB. In the documentation, warn that the cells must be protected, or charged outside the robot with a balancing 2S charger.

---

## 6. Main ESP32: ESP32-S3-DevKitC-1 N16R8

- **Confirmed board:** ESP32-S3-DevKitC-1 N16R8 (ESP32-S3-WROOM-1 module, 16 MB flash, 8 MB octal PSRAM), 44 pins in 2 rows of 22.
- **It may be a commercial clone.** Before drawing the footprint:
  - Compare the clone's pin silkscreen with Espressif's official pinout.
  - Measure the length, the width, the row spacing and the position of the USB connectors and the antenna.
  - If anything does not match, **stop and ask for photos**.
- **Mounting:** 2 × 1×22 female headers, so the board is removable.
  - At least one USB-C connector must be reachable from an edge. The board usually has two: UART and native USB.
  - The RESET and BOOT buttons must stay reachable.
  - The antenna must stick out past the PCB edge, or have a copper-free area under it.
- **Pins that must NOT be used** (verify with Espressif's documentation):
  - **GPIO35, GPIO36, GPIO37:** on octal-PSRAM boards (like the N16R8) the internal memory uses them, so they are unavailable.
  - Strapping: GPIO0, GPIO3, GPIO45, GPIO46.
  - Native USB: GPIO19, GPIO20.
  - UART0 (console and programming): GPIO43, GPIO44.
  - On-board RGB LED: GPIO48 on revision v1.0 and GPIO38 on v1.1. Check which one the clone has and avoid it.
- **Candidate signal pins to verify:** GPIO1, 2, 4–18, 21, 39–42 and 47.
- **Required signals (17 GPIOs):**
  - TB6612: AIN1, AIN2, PWMA, BIN1, BIN2, PWMB, STBY (7 outputs).
  - XSHUT_FRONT, XSHUT_LEFT, XSHUT_RIGHT (3 outputs).
  - I2C: SDA, SCL.
  - UART to the ESP32-CAM: TX, RX.
  - LED_FRONT, LED_LEFT, LED_RIGHT (3 outputs).
- **Grouping:** group signals by area to simplify routing:
  - Motor signals near the TB6612.
  - I2C and XSHUT toward the sensor connectors.
  - LED signals toward their connectors.
- **Bluetooth:** the ESP32-S3 has only Bluetooth Low Energy (BLE 5), no Classic Bluetooth. Note it for the firmware: the Xbox controller must connect over BLE. No extra hardware is needed.
- **Power:** through its 5V pin, via the Schottky diode (§5.3).

### 6.1 Pin mapping (revised for the single-layer PCB, implemented in the schematic)

On a single copper layer two tracks cannot cross. The pins were **reassigned** so that the signals leave the ESP32 headers in the same order as their destinations. The firmware does not exist yet, so the change costs nothing. This **replaces** the earlier mapping (SDA GPIO8, SCL GPIO9, XSHUT 4/5/6, LEDs 10/11/12, motors on 1/2/42/41/40/39/21).

| Signal | GPIO | Header pin | Reason |
|---|---|---|---|
| SDA | GPIO4 | J1-4 | First of the sensor group, so the bus can run as a staircase without crossings |
| SCL | GPIO5 | J1-5 | |
| XSHUT_RIGHT | GPIO6 | J1-6 | Order RIGHT, FRONT, LEFT matches the connectors from top to bottom |
| XSHUT_FRONT | GPIO7 | J1-7 | |
| XSHUT_LEFT | GPIO15 | J1-8 | |
| UART_ESP_TO_CAM | GPIO17 | J1-10 | Default `Serial1` TX1 (unchanged) |
| UART_CAM_TO_ESP | GPIO18 | J1-11 | Default `Serial1` RX1 (unchanged) |
| LED_FRONT | GPIO8 | J1-12 | |
| LED_LEFT | GPIO9 | J1-15 | |
| LED_RIGHT | GPIO10 | J1-16 | |
| PWMB | GPIO1 | J3-4 | The TB6612FNG sits right behind J3 with its pins in the opposite order, so the signals are listed in reverse |
| BIN2 | GPIO2 | J3-5 | |
| BIN1 | GPIO42 | J3-6 | |
| STBY | GPIO41 | J3-7 | R2 10 kΩ to GND keeps the motors off while the ESP32 boots |
| AIN1 | GPIO40 | J3-8 | |
| AIN2 | GPIO39 | J3-9 | |
| PWMA | GPIO21 | J3-18 | |

- **Firmware note:** I2C is not on the Arduino default pins. Use `Wire.begin(4, 5)`.
- **Unused on purpose:**
  - 0, 3, 45, 46 (strapping).
  - 19, 20 (USB).
  - 43, 44 (console).
  - 35, 36, 37 (octal PSRAM).
  - 38, 48 (RGB LED).
  - 47.
- **Spare pins:** GPIO11, 12, 13, 14 and 16 (J1-9, 17, 18, 19, 20).

---

## 7. ESP32-CAM and external camera

- **Module:** AI Thinker ESP32-CAM (about 27 × 40.5 mm; verify), on the same PCB.
- **Mounting:** 2 × 1×8 female headers, so it can be removed easily and programmed on the ESP32-CAM-MB base.
  - Mark the orientation on the silkscreen (5V pin and antenna side) so it cannot be inserted backwards.
- **UART pins:** no microSD is used.
  - UART to the main ESP32: **IO13 = camera RX** (`UART_ESP_TO_CAM`) and **IO14 = camera TX** (`UART_CAM_TO_ESP`).
  - They do not interfere with the camera, boot pins (GPIO0, 2, 12, 15), the flash LED (GPIO4) or the PSRAM (GPIO16).
  - Do not use GPIO1/GPIO3 (U0) for this link.
  - IO14 may output a few garbage bytes at boot; the firmware must discard them.
- **Unconnected pins:**
  - All other pins stay unconnected.
  - The module's 3V3 and VCC pins are **outputs** and are not connected.
- **Power:** through the 5V pin, with local capacitors (§13).
- **External camera:**
  - The stock OV2640 is replaced by one with a long flex cable (24 pins, 0.5 mm pitch), fixed at the front of the chassis.
  - Use the shortest flex that reaches, ideally 10 cm or less. Long cables can corrupt the image; if that happens, lower XCLK in firmware (for example from 20 to 10 MHz). Document this.
  - Place the ESP32-CAM with its FPC connector facing the front of the robot.
  - Keep the flex path clear, with room to lift the connector latch by hand.
  - Route the flex away from the TB6612, the motor traces and the motor wires.
  - Explain how to fix the flex to the chassis with strain relief (printed holder or tape) and how the enclosure must keep the field of view clear.
- **Antennas:** copper-free area under the ESP32-CAM antenna. Put the ESP32-S3 and ESP32-CAM antennas on different PCB edges, as far apart as possible, because both use 2.4 GHz.

---

## 8. UART between ESP32 and ESP32-CAM

- Direct traces on the PCB, no connector.
- Net names follow the signal direction:
  - `UART_ESP_TO_CAM`: ESP32 TX → camera RX.
  - `UART_CAM_TO_ESP`: camera TX → ESP32 RX.
- Both sides use 3.3 V logic, so no level shifter is needed.
- Route the UART away from the motor traces. Suggested firmware speed: 115200 baud.

---

## 9. VL53L0X sensors (off-board)

- **Modules:** 3 GY-530 modules, mounted in the concave front area of the chassis.
  - Verify the pins of the real module (typically VIN, GND, SCL, SDA, GPIO1, XSHUT).
  - GPIO1 is not used.
- **Power:** **3.3 V**, so the module's I2C pull-ups can never put 5 V on the ESP32.
- **Connectors:** one 5-pin JST-XH per sensor: `SENSOR_FRONT`, `SENSOR_LEFT`, `SENSOR_RIGHT`.
  - Same order on all three: 1 3V3, 2 GND, 3 SDA, 4 SCL, 5 XSHUT.
  - On the PCB all three sit on the front edge, stacked RIGHT (top), FRONT, LEFT (bottom) with the pins running away from the edge (pin 1 nearest the edge). This order lets the SDA, SCL and 3V3 buses run without crossing.
- **I2C bus:** one shared bus (SDA on GPIO4, SCL on GPIO5, see §6.1), 100 kHz recommended, cables of 25 cm or less.
- **XSHUT:** RIGHT on GPIO6, FRONT on GPIO7, LEFT on GPIO15.
- **Pull-ups:**
  - Check the ones on the modules (three in parallel).
  - **Decision (user):** no extra pull-ups on the PCB. The VL53L0X GY-530 modules already include their pull-ups, so R3 and R4 were removed from the design.
- **XSHUT level:** check what voltage the module's pull-up takes XSHUT to, and that the ESP32 can drive it safely. Recommended: "enable" a sensor by setting its pin as an input so the module pull-up raises it, instead of driving a logic 1.
- **Firmware startup sequence:**
  1. Drive all XSHUT pins low.
  2. Enable one sensor.
  3. Assign it a new address (0x30, 0x31, 0x32; the default is 0x29).
  4. Repeat with the next sensor.
- **Field of view:** each VL53L0X sees a cone of about 25°. With sensors at 0° and ±45°, there are blind spots between the cones and on the pure sides. This is accepted; document it.
- **Expected behavior:** when the ball is inside the concave area, the FRONT sensor detects it and the red LED turns on. Document this.

---

## 10. Proximity LEDs (off-board, university rules)

- The 3 proximity LEDs are driven **only by the proximity sensors**. Each LED shows the detection of its own sensor.
- **The university rules set the colors, and they cannot be changed:**

| Sensor | LED | Color (rules) | PCB connector |
|---|---|---|---|
| FRONT | LED_FRONT | **Red** | J_LED_FRONT |
| RIGHT | LED_RIGHT | **Green** | J_LED_RIGHT |
| LEFT | LED_LEFT | **Yellow** | J_LED_LEFT |

- **Activation rule:**
  - The LED is on when its sensor reads **≤ 100 mm (10 cm)**.
  - The LED is off when the reading is > 100 mm, invalid or out of range.
  - The firmware may average several readings to avoid flicker. It must not use hysteresis that keeps a LED on above 100 mm, because that would break the rules.
- **Location:** the LEDs are **off the PCB**, mounted on the enclosure where they are visible, as the rules require.
- **Connection:** each LED has its own 2-pin male header, 2.54 mm pitch (`LED+` / `LED−`), connected with Dupont jumpers.
  - One connector per LED, to avoid mix-ups.
  - Silkscreen with the name, color and polarity (`+` on the anode).
- **Resistors:** one per LED, on the PCB, sized from each color's Vf for about 2–5 mA at 3.3 V.
  - The current value is 470 Ω, which gives about 2.5–3 mA.
  - **R5 (green) is provisional:** it suits a classic green (Vf ≈ 2.1 V). A bright green (Vf ≈ 3 V) needs a recalculation.
- **Documentation note:** a LED connected backwards simply does not light and is not damaged. Just flip the connector.
- **Power LED:** this is a different LED from these three. It is blue, off-board and sits next to the switch (§5.4).

---

## 11. TB6612FNG and motors

- **Module:** commercial TB6612FNG breakout (SparkFun "Dual TB6612FNG 1A" type or a clone), **plugged into the PCB** through female headers.
  - All of its connections go through the PCB: power, signals and motor outputs.
  - The motors never connect directly to the module.
- **Before drawing the footprint:** verify the exact pin order of the purchased module, because clones vary. If there is no data, stop and ask for a photo. The schematic currently uses the SparkFun order (pin 1 = PWMA, pin 4 = STBY).
- **Connections:**
  - VM ← `+VBAT_SW`.
  - VCC ← `+3V3`.
  - All module GND pins to GND.
  - AIN1, AIN2, PWMA, BIN1, BIN2, PWMB, STBY ← ESP32.
  - Channel A outputs → MOTOR_LEFT; channel B outputs → MOTOR_RIGHT.
- **Motor connectors:** `MOTOR_LEFT` and `MOTOR_RIGHT` are 2-pin screw terminals (5.08 mm pitch) on the left and right edges, above each motor, so the wires run straight up from the bottom level.
  - Document that if a motor spins backwards, swap its wires or invert it in firmware.
- **Motor capacitor:** a 100 nF ceramic soldered across each motor's terminals, on the motor itself, not on the PCB.
- **Wheels:** check that the 50 mm wheels fit the TT motor shaft.

---

## 12. External connection summary

Every off-board component connects to the PCB through a labeled connector. Nothing is soldered directly to the PCB or to the modules.

In the schematic, reference designators end in a digit (`J_BAT1`, `J_SENSOR_FRONT1`…) because KiCad requires it; otherwise it renumbers them at PCB stage.

| Reference | Off-board part | Connector type | Signals | Suggested location |
|---|---|---|---|---|
| J_BAT1 | 2S batteries | XT30 male vertical (AMASS XT30UPB-M) | 1 BAT− (chamfered side), 2 BAT+ (flat side), matching the KiCad footprint's `−`/`+` marks | Near where the battery wires come up |
| J_SWITCH1 | Switch | 2P screw terminal, 5.08 mm, ≥3 A | SW_BAT, SW_OUT | Near J_BAT1 |
| J_LED_POWER1 | Power LED (blue) | 2P male header, 2.54 mm | LED+, LED− | Near J_SWITCH1 |
| J_MOTOR_LEFT1 | Left motor | 2P screw terminal, 5.08 mm | M_L1, M_L2 | Left edge |
| J_MOTOR_RIGHT1 | Right motor | 2P screw terminal, 5.08 mm | M_R1, M_R2 | Right edge |
| J_SENSOR_FRONT1 | Front sensor | JST-XH 5P vertical | 3V3, GND, SDA, SCL, XSHUT_FRONT | Front edge |
| J_SENSOR_LEFT1 | Left sensor | JST-XH 5P vertical | 3V3, GND, SDA, SCL, XSHUT_LEFT | Left / front edge |
| J_SENSOR_RIGHT1 | Right sensor | JST-XH 5P vertical | 3V3, GND, SDA, SCL, XSHUT_RIGHT | Right / front edge |
| J_LED_FRONT1 | Red LED | 2P male header, 2.54 mm | LED+, LED− | Front edge |
| J_LED_LEFT1 | Yellow LED | 2P male header, 2.54 mm | LED+, LED− | Left edge |
| J_LED_RIGHT1 | Green LED | 2P male header, 2.54 mm | LED+, LED− | Right edge |
| ESP32-CAM FPC | OV2640 camera | 24-pin flex (on the module itself) | Camera bus | Toward the front |

**Plug-in modules on the PCB:**

| Module | Connection |
|---|---|
| ESP32-S3-DevKitC-1 | 2 × 1×22 female headers (`U_ESP_J1`, `U_ESP_J3`) |
| ESP32-CAM | 2 × 1×8 female headers (`U_CAM1`, `U_CAM2`) |
| TB6612FNG | 2 × 1×8 female headers (`U_TB1`, `U_TB2`) |
| MP1584 | 4 pins soldered to the PCB (`U_MP1584`); verify against the real module |

**Connector rules:**

- **Power connectors** (battery, switch, motors): screw terminals or XT30, rated for the real current.
- **Low-current signals** (LEDs): 2.54 mm headers with Dupont wires.
- **Sensors:** polarized JST-XH, so 3V3 and GND cannot be swapped.
- **Placement:** all connectors on the edges, facing where their cable comes up, with room to plug and unplug by hand.

---

## 13. Capacitors

| Location | Value | Minimum voltage |
|---|---|---|
| `+VBAT_SW`, next to J_SWITCH | 470 µF low-ESR electrolytic + 100 nF X7R | 16 V (25 V preferred) |
| TB6612 VM, right at the header | 220 µF electrolytic + 100 nF | 16 V (25 V preferred) |
| MP1584 input | 22 µF + 100 nF | 16 V (25 V preferred) |
| MP1584 output (`+5V`) | 100 µF electrolytic + 10 µF ceramic | 10 V |
| ESP32 5V pin (after the diode) | 47–100 µF + 100 nF | 10 V |
| ESP32-CAM 5V pin | 470 µF electrolytic + 10 µF + 100 nF | 10 V |
| `+3V3` next to TB6612 VCC | 10 µF + 100 nF | 10 V |
| Each sensor connector | 10 µF + 100 nF | 10 V |
| Each motor's terminals (on the motor) | 100 nF ceramic | 50 V |

- Use X7R or X5R ceramics in 0805 or larger, and account for capacitance loss with DC bias.
- Place each 100 nF as close as possible to the pin it decouples.
- Check that the electrolytics' height fits the enclosure.

---

## 14. PCB design rules

- **Decision (user, latest):** **one copper layer.** The copper is `B.Cu` (the bottom face). `F.Cu` has **no copper**: a keepout rule area (`NO_COPPER_TOP`) covers the whole top face, so only the through-hole pads show on top. Reason: most local fabs do not plate the holes, and the user does not want to plate vias by hand. A single layer needs no plated holes at all.
  - This replaces the earlier 2-layer decisions ("no plane, tracks only", then "GND plane on B.Cu and signals on F.Cu").
  - The KiCad file still declares two copper layers, because KiCad cannot keep only `B.Cu`. Plot or print **only `B.Cu`** for etching.
- **GND is a pour on the same layer (`B.Cu`).** It fills every free area. Pads connect to it through thermal reliefs (gap 0.3 mm, spokes 0.6 mm), so they can still be hand-soldered. Where a track would cut a GND pad off from the pour, a short GND track connects that pad (10 short GND segments in the final board).
- **Decision (user, confirmed):** **all components are through-hole (THT)** so they can be soldered easily by hand. No SMD parts. Components go on the **top face**; their legs are **soldered from below**, on the copper side.
- **No vias and no jumper wires.** The pin mapping (§6.1) and the placement were chosen so that no two tracks cross.
- 1.6 mm FR4, 1 oz copper, **home-made board rules (user):**
  - Tracks **≥ 0.4 mm**. Tracks that pass **between pads** may be **0.3 mm**. KiCad refuses anything under 0.3 mm (`auxi.kicad_pro`).
  - Clearance 0.2 mm between copper objects, and 0.5 mm from copper to the board edge.
  - Smallest hole **0.3 mm**. Default via 1.1 mm pad / 0.3 mm hole (not used, but set so any future via follows the rule).
  - **Pad diameter = drill diameter + 0.8 mm** on every pad hole (0.4 mm of copper all around). It is enforced by the custom DRC rule `pad_ring_0.4mm` in `auxi.kicad_dru`. Four standard footprints were copied into `auxi.pretty/` with bigger pads (PinSocket 1×8, PinSocket 1×22, PinHeader 1×2, JST-XH B5B-XH-A); the schematic points to the copies.
- **Trace widths used** (verify the power ones with an IPC-2152 calculator using the stall current, not the nominal current):
  - Signals: 0.4 mm, or 0.3 mm where they squeeze between pads or run in a tight bundle (55 segments, 377 mm in total).
  - `+3V3`: 0.3–0.4 mm (about 100 mA).
  - `+5V` and `+5V_ESP`: 0.6 mm.
  - Battery, J_SWITCH, VM and motors: 1.2 mm.
- **Zones:**
  - Power zone: XT30, J_SWITCH, MP1584, TB6612 and motor terminals (rear and rear-left of the robot).
  - Logic and radio zone: ESP32, ESP32-CAM, sensor and LED connectors.
- **Ground and routing:**
  - Keep motor currents away from I2C, UART, the ESP32-CAM and the antennas.
  - Tracks may pass between the 2.54 mm pads (the gap is 0.74 mm), but only at 0.3 mm width.
  - Small parts (capacitors, R2) sit under the TB6612 module and the ESP32 USB end, where there is room. **Electrolytic capacitors must not go under a module** (they are taller than the 8.5 mm of free height).
- Keepouts at the M3 holes and under both antennas (`ANT_ESP32`, `ANT_CAM`): no tracks, vias or pour.
- **Bottom side:** it is now the copper side. It has no components. Only through-hole solder joints, with the §4.3 clearance to the batteries.

---

## 15. Silkscreen

> **Later stage (PCB).** Do not execute now; keep it in mind when choosing footprints.

Label clearly:

- ESP32-S3, ESP32-CAM, TB6612FNG, MP1584.
- BAT+ and BAT−.
- SWITCH / ON-OFF and POWER LED.
- MOTOR_LEFT and MOTOR_RIGHT.
- SENSOR_FRONT, SENSOR_LEFT and SENSOR_RIGHT.
- LED_FRONT (RED), LED_LEFT (YELLOW) and LED_RIGHT (GREEN).
- Pin 1 and polarity on every connector and capacitor.
- Module orientation, and a "5.0 V" setting note next to the MP1584.
- A "FRONT" arrow showing the front of the robot.
- Project name and version ("auxi v1").

Do not place text on pads or in areas covered by the modules once mounted.

---

## 16. ERC and DRC

- **ERC:**
  - Fix real errors. Never ignore or suppress errors to get a clean-looking result.
  - Document the reason for any warning that remains.
  - Use power symbols and `PWR_FLAG`s only where they belong, and explain why.
  - Current `PWR_FLAG`s: `+VBAT_SW`, `GND` and `+5V_ESP`. These nets are fed only through connectors or the diode, which KiCad treats as passive pins.
- **DRC:** runs at the PCB stage.

---

## 17. Final mechanical verification

> **Later stage (PCB).** Do not execute now; keep it in mind when choosing footprints.

- The PCB is 110 × 140 mm and fits the top level (still to be confirmed with the real chassis; see §20).
- Standoff height keeps the PCB above batteries, motors and cables, with the minimum clearance to metal contacts.
- The PCB does not hit the wheels or block the concave area or the camera.
- 4 M3 holes with keepouts.
- ESP32-S3 USB and buttons are reachable.
- ESP32-CAM is removable, with access to the FPC connector latch.
- The camera flex path is clear and away from the power zone.
- All off-board connectors are reachable and face where their cables come up: battery, switch, power LED, motors, sensors and LEDs.
- The enclosure and chassis have a planned cable path.
- No collisions between components, modules, cables and enclosure.

---

## 18. Stages and execution order

### Stage 1: schematic (DONE, pending verification)

1. Review this specification and resolve with the user the open items marked **needed for the schematic** (§20).
2. Present the full ESP32-S3 and ESP32-CAM pin mapping, the current budget, and the switch and wire selection. **Wait for approval.**
3. Create the KiCad project `auxi` through the MCP.
4. Create the schematic, organized in clearly titled blocks:
   - Power: battery, switch, power LED, MP1584, diode and capacitors.
   - ESP32-S3.
   - ESP32-CAM.
   - TB6612FNG and motors.
   - Sensors.
   - Proximity LEDs.
5. Use net labels with the names from this file (`+VBAT_SW`, `+5V`, `+5V_ESP`, `+3V3`, `UART_ESP_TO_CAM`, `UART_CAM_TO_ESP`, `SDA`, `SCL`, `XSHUT_FRONT`, …).
6. Assign and verify every symbol's footprint.
7. Run ERC and fix.
8. Re-check power, connectors and signals against this file.
9. Generate the preliminary BOM and schematic documentation.
10. Present a final summary with any remaining warnings, then **stop**.

**Current state:**

- The schematic is complete, and ERC passes with 0 errors and 0 warnings.
- The pin mapping is approved.
- The design is **preliminary** until the §20 items are closed against the real modules.

### Stage 2: PCB (DONE as a single-layer 110 × 140 mm board, pending the real-part checks)

The user had no calipers, so they approved using common web dimensions (§4.4). Every module position is **provisional** until checked against the real part.

**History.** The first PCB was 90 × 100 mm with a GND plane on `B.Cu` and signals on `F.Cu` (2 layers, 75 vias). Because local fabs do not plate holes, the user asked for a **single-layer board with a GND pour**, soldered from below, **110 × 140 mm**, with the track and hole rules of §14. The board was re-placed and re-routed from scratch. The old 2-layer version is in the Git history (before this change).

**Board orientation (assumed from the user's sketch):**
- Board outline: **x 106.07–216.07 mm, y 47.5–187.5 mm** in KiCad coordinates (110 × 140 mm, 2 mm corner radius).
- **Left edge (x = 106.07) = robot FRONT** (concave area).
- **Top edge (y = 47.5) = robot RIGHT side**, **bottom edge (y = 187.5) = robot LEFT side**, right edge (x = 216.07) = REAR.
- The 110 mm side runs front to back; the 140 mm side runs across the robot.

**Placement (x, y of pin 1 in mm):**

| Part | Position | Notes |
|---|---|---|
| ESP32-S3 | `U_ESP_J1` (156.0, 122.17), `U_ESP_J3` (178.86, 122.17) | Pins run toward the bottom edge. **USB end about 5 mm from the bottom edge** (module end at y ≈ 182.8, board edge at 187.5), so the USB-C plug is reachable and overhangs the edge. Antenna keepout `ANT_ESP32` x 154.5–180.5, y 112.5–121.5 |
| ESP32-CAM | `U_CAM1` (150.0, 166.43), `U_CAM2` (150.0, 143.57) | Pins at the rear end of the module, antenna and flex toward the front. Keepout `ANT_CAM` x 106.6–119.1, y 141–169 |
| TB6612FNG | `U_TB1` (185.0, 145.03), `U_TB2` (202.78, 145.03), rotated 180° | Right behind the ESP32-S3 J3 row. Control row toward the ESP32, power row toward the rear |
| MP1584 | centre (192.5, 161.5), rotated 270° | Below the TB6612. IN+ toward the TB6612, OUT+ toward the bottom edge |
| Sensors | `J_SENSOR_RIGHT1` (110.5, 95.0), `FRONT1` (110.5, 109.0), `LEFT1` (110.5, 123.0) | On the front edge, pins running along +x |
| Proximity LEDs | headers at x = 127, y = 174.5 / 179.5 / 184.5 (FRONT, LEFT, RIGHT), resistors R3–R5 at x = 116 | Lower-left corner |
| Motors | `J_MOTOR_RIGHT1` (209.5, 112.0), `J_MOTOR_LEFT1` (209.5, 156.0) | Rear edge, wire entry toward the edge |
| Power | `J_SWITCH1` (209.5, 176.0), `J_BAT1` (192.0, 182.5), `J_LED_POWER1` (198.0, 62.0) | `J_LED_POWER1` is not on the corner on purpose: the +VBAT_SW copper that feeds it must not close a loop with the board edge, or the `+3V3` track to the TB6612 VCC pin would have no way through on a single layer |
| M3 holes | 5 mm from each corner | |

- **Why the ESP32 is where it is (user request).** The ESP32-S3 USB end is at the board edge (≈ 5 mm) so the cable can be plugged in. The strip between the module end and the edge (≈ 5 mm) carries the `+5V` track, D1 and a few small parts, which sit under the overhanging module (they are short). C9 is the low-profile capacitor next to that area.
- Big capacitors C1, C2, C3, C5, C6 sit in a column at x ≈ 188 north of the TB6612 (next to the +VBAT_SW track); C11 is in the lower-left. The sensors are spread along the front edge (14 mm pitch) and the LED headers are in the lower-left corner.
- Silkscreen: module outlines for the ESP32-S3 (end lines), ESP32-CAM and MP1584 (its own footprint), TB6612 outline on `F.Fab`, and the labels of §15. Reference designators stay on `F.Fab`.

**Routing method.** The MCP KiCad tools were not available, so the work was done with scripts (KiCad 10 Python API) and **Freerouting 2.1.0**:
1. The placement was generated by a script (fixed module positions plus a greedy search for the passive parts).
2. Freerouting routed the board **on one layer only**, with GND routed as a normal net, 0.2 mm clearance and 0.3 mm base track width.
3. The GND tracks that were not needed were removed one by one, as long as the pour still reached every GND pad (about 120 → 10 short GND segments). 
4. Every track that could be widened to 0.4 mm without breaking the 0.2 mm clearance was widened.
- The scripts are not part of the repository. Re-running the layout means repeating this procedure in KiCad (or with Freerouting).

**Routing result (final board):**

| Item | Value |
|---|---|
| Tracks | 199, all on `B.Cu`, 0 vias, 0 jumper wires |
| Widths | 0.3 mm × 41 (209 mm), 0.4 mm × 47, 0.6 mm × 69, 0.8 mm × 10 (GND), 1.2 mm × 32 |
| Net classes | `PWR_BAT` 1.2 mm (`+VBAT_SW`, `SW_BAT`, `M_*`), `PWR_LOW` 0.6 mm (`+5V`, `+5V_ESP`), `GND` 0.8 mm, `Default` 0.4 mm |
| Pour | `GND_PLANE` on `B.Cu`, one zone, thermal reliefs, no isolated islands |
| Drills | 0.8, 0.95, 1.0, 1.1, 1.3, 1.8 mm (pad holes) and 4 × 3.2 mm (M3) |
| DRC (`kicad-cli pcb drc`, zones refilled, custom rules) | **0 violations, 0 unconnected items** |
| ERC | 0 errors, 0 warnings |
| Schematic parity | Same result as before this change (net-name prefix and field warnings only; the PCB nets match the schematic pin by pin) |

**Things to review by eye before etching (DRC cannot judge them):**
- The `+3V3` track to the TB6612 VCC pin is long (0.6 mm wide): it goes along the board's upper area and the east side because a single layer cannot cross the +VBAT_SW and signal tracks. Check it for shorts or breaks when etching.
- The 0.3 mm tracks (I2C, UART, LED signals) run in tight bundles with 0.2 mm gaps. They are the most likely to break or short when etching. Print the copper page, look at it under light, and consider a slightly longer exposure.
- I2C/UART tracks versus the motor tracks and the antenna keepouts (§14). The motor tracks are all in the rear part of the board.
- Any 0.3 mm track that passes **between header pads** (the pads are 1.8 mm wide on a 2.54 mm pitch, so each gap is 0.74 mm and a 0.3 mm track keeps 0.22 mm of clearance on each side). Look at the ESP32-S3 and TB6612 header rows.
- Capacitor heights (§14) and the free space above the ESP32 USB end.

**Pending:** the 2D mechanical check with real measurements (§4.3/§17), and the open items in §20.

### Later stages

- Mechanical verification (§17) once the real parts are measured.

---

## 19. Deliverables and documentation

**Stage 1 deliverables:**

- KiCad project `auxi` (`.kicad_pro`) with the complete schematic (`.kicad_sch`) and a passing ERC.
- Assigned and verified footprints. Custom libraries only when needed, and verified against the real part.
- ESP32-S3 and ESP32-CAM pin mapping tables.
- Preliminary BOM, split into two lists:
  - **PCB components:** `docs/auxi_bom_pcb.csv`, exported from the schematic.
  - **Off-board components:** the table below.

**Off-board BOM:**

| Qty | Description |
|---|---|
| 1 | Switch ≥ 5 A DC and ≥ 12 V DC, **verified in its datasheet** (panel-mount mini rocker). Confirm it against the measured stall current. |
| 1 | 5 mm blue LED (power) |
| 1 + 1 + 1 | 5 mm LEDs: red, yellow and green (see the R5 note in §10) |
| 3 | VL53L0X GY-530 modules |
| 3 | 5-pin JST-XH cables, ≤ 25 cm (with XHP-5 female housing) |
| 4 × 2 | Female Dupont jumpers for the 4 LEDs |
| 1 | OV2640 camera with a long 24-pin, 0.5 mm pitch flex (≤ 10 cm if possible) |
| 2 | TT gear motors + 2 × 50 mm wheels + 1 caster |
| 2 | 100 nF 50 V ceramic capacitors for the motor terminals |
| 1 | 2×18650 battery holder + 2 protected 18650 cells + female XT30 cable |
| — | 20 AWG wire (battery and switch) and 22 AWG wire (motors) |
| 4 | M3 standoffs + screws (length defined at PCB stage) |

**Plain-language documentation (`README.md` and this file):**

- Architecture, power, pinout, off-board connectors (what goes where), sensors, rule-compliant LEDs, motors, TB6612FNG, UART, external camera and key decisions.

**Assembly warnings (must appear in `README.md`):**

1. Set the MP1584 to 5.0 V with a multimeter **before** plugging in any module.
2. Check battery polarity.
3. The switch path carries all of the robot's current: use the specified wire gauge (20 AWG) and screw terminal, never Dupont jumpers.
4. Remove the ESP32-CAM to program it.
5. Motor PWM must never exceed 70 %.
6. 18650 cells must be protected, or charged with a balancing 2S charger.

The project is **not** finished just because a `.kicad_sch` file exists: it must pass ERC and the electrical review.

---

## 20. Open items (stop and ask if unresolved)

**Needed to finalize the schematic:**

1. Confirm that the purchased ESP32-S3-DevKitC-1 N16R8 matches the official pinout. If it is a clone, ask for photos of both sides. Also confirm its RGB LED revision and its row spacing.
2. Real pinout of the purchased TB6612FNG (SparkFun order assumed), MP1584 (footprint `auxi:MP1584_Mini_PROVISIONAL` has estimated pads) and GY-530 (XSHUT level; the pull-ups are already on the module, so R3/R4 were removed).
3. Real TT motor stall current, to validate the driver, switch and power connectors.
4. Green LED Vf, to set the final value of R5.
5. ESP32-CAM silkscreen, to confirm the pin order used.
6. ~~Which XT30UPB-M pin is `+`~~ **Resolved:** pad 2 = `+` (flat side), pad 1 = `−` (chamfered side). The schematic was corrected to match.
7a. Caliper check of row spacing (the layout depends on these, and the pads are fixed to them): ESP32-S3 clone (22.86 mm assumed), ESP32-CAM (22.86 mm assumed), TB6612 red module (17.78 mm assumed), and MP1584 hole positions (estimated from a photo). Also confirm which end of the ESP32-CAM has the header pins.

**Needed for the PCB (later stages):**

7. Does the 150 × 150 mm chassis include the wheels?
8. Orientation of the 110 × 140 mm PCB in the chassis (the layout assumes 110 mm front to back) and whether it clears the wheels and the concave area.
9. Real measurements of the TT motor, battery holder and 50 mm wheels, and the height available on the bottom level.
10. Location of the proximity LEDs on the enclosure, per the rules.
11. Camera flex length, based on the final ESP32-CAM position.
12. Height of the electrolytic capacitors that will be bought (C1, C3, C11), and a low-profile 6.3 × 7 mm part for C9 (it sits next to the ESP32 USB plug area).

---

## 21. Repository files

| Path | Notes |
|---|---|
| `auxi.kicad_pro` / `auxi.kicad_sch` / `auxi.kicad_pcb` | Project, schematic, PCB (single layer, 110 × 140 mm, routed) |
| `auxi.kicad_dru` | Custom DRC rules: pad ring ≥ 0.4 mm, hole ≥ 0.3 mm |
| `auxi.kicad_sym`, `auxi.pretty/` | Project libraries, loaded through `sym-lib-table` and `fp-lib-table` with `${KIPRJMOD}` |
| `auxi.pretty/MP1584_Mini_PROVISIONAL.kicad_mod` | **Estimated pads.** Verify against the real module before etching |
| `auxi.pretty/PinSocket_1x08…`, `PinSocket_1x22…`, `PinHeader_1x02…`, `JST_XH_B5B-XH-A…` | Copies of the standard footprints with bigger pads (pad = drill + 0.8 mm). The schematic uses these |
| `docs/auxi_esquematico.pdf`, `docs/auxi_bom_pcb.csv` | Regenerate after schematic changes |
| `docs/auxi_pcb_real_scale.pdf` | 3 pages at 1:1 (letter): top view, `B.Cu` copper, `B.Cu` mirrored. Regenerate after PCB changes |
| `docs/auxi_pcb_top.png`, `docs/auxi_pcb_bottom.png`, `docs/auxi_pcb_placement.png` | Renders. Regenerate after PCB changes |
| `auxi_drc_violations.json` | Last DRC report |
| `auxi.kicad_prl` | Local view settings; gitignored |

**Module representation in the schematic:**

- Each header row of a module is its own symbol with a standard female-header footprint.
- There is no official published row spacing and clones vary, so the rows are placed at PCB stage using caliper measurements.
- Pin **numbers** match the physical header position, while the symbol graphics group pins by function.
