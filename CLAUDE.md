# CLAUDE.md

Guidance for Claude Code when working in this repository. This file is the **complete design specification** for the project. No external document is needed.

---

## 1. Project overview

**auxi** is a small soccer robot, a first-semester university project. This repo holds the KiCad design for its **single custom PCB**.

- **Drive:** 2 driven wheels (yellow TT gear motors) plus 1 rear caster wheel.
- **Chassis:** 3D-printed, **v8** (see §4). About 144 mm long (limit 150 mm). The front is a **printed V (07) or curved (08) piece that guides the ball**; it is a swappable part bolted to the base.
- **Two-level architecture:**
  - **Bottom level (chassis):** base, tray, motors, caster, battery (at the rear, behind a door), ballast and wiring.
  - **Top level:** a **single PCB** on 4 printed columns (M3 × 25 screws), 10 mm above the tray and above the batteries and motors.
- **The PCB carries:**
  - The plug-in modules: ESP32-S3-DevKitC-1, ESP32-CAM and the TB6612FNG breakout.
  - The MP1584 regulator.
  - The passive components.
  - **All connectors.**
  - **The 4 LEDs, soldered directly on the PCB**: the 3 proximity LEDs and the green power LED (see §10).
- **Everything else is off-board** and connects to the PCB through connectors:
  - The 3 VL53L0X sensors.
  - The switch (screw terminal, in the switch tower).
  - The motors (screw terminals).
  - The battery (screw terminal).
  - The camera (standard 24-pin module on the camera arch, plugged into the ESP32-CAM through its ~2 cm flex).
- **Main ESP32:** controls the robot.
  - Receives commands from an **Xbox controller over Bluetooth Low Energy**.
  - Reads the 3 sensors.
  - Drives the TB6612FNG and the LEDs.
- **ESP32-CAM AI Thinker:**
  - Streams live video over Wi-Fi to a phone.
  - Shows the distances it receives from the main ESP32 over UART.
  - Its OV2640 camera sits in a printed cradle on the camera arch, at the front of the chassis (see §7).
- **Sensors:** 3, mounted on printed supports (04) at the front: FRONT (0°), LEFT (+45°), RIGHT (−45°).
- **Proximity LEDs:** 3, soldered on the PCB. Their behavior depends on the sensors and follows the university rules (§10).

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
| PCB location | Top level, on 4 printed columns with M3 × 25 screws (printed spacers keep it 10 mm above the tray), above the batteries and motors. |
| Enclosure | **None** (user decision): the robot has no enclosure; the PCB stays visible on top of the chassis. So the LEDs are visible (university rules) and no windows or enclosure clearances are needed. |
| PCB size | **110 × 140 mm**, set by the user (it was 90 × 100 mm; the extra room is for the single-layer routing). Do not enlarge it without approval. **The 110 mm side is the FRONT of the robot (between the wheels) and the 140 mm side runs front to back. Confirmed by the user with a photo of the chassis.** |
| On the PCB | ESP32-S3-DevKitC-1, ESP32-CAM, TB6612FNG module, MP1584 module, capacitors, resistors, diode and connectors. |
| Off-board (with a connector on the PCB) | 3 VL53L0X sensors, switch, 2 motors, battery and OV2640 camera. |
| Chassis front | Printed V (07) or curved (08) front piece that guides the ball (recommended: the curved one, 10 mm deep against 12.9 mm maximum). It is bolted to the base on pivots. |
| Battery | 2 × 18650 in series (2S): 6.0 V minimum, 7.4 V nominal, 8.4 V maximum. At the rear of the bottom level, behind a door (14), so it can be charged without removing the PCB. |
| Battery connector | **2-pin 5.08 mm screw terminal** (same type as the motors), soldered into the pads of `J_BAT1`, which still have the XT30UPB-M footprint (5.0 mm pitch, 1.8 mm holes). BAT− on the left pad (`-` mark), BAT+ on the right pad (`+` mark). **There is no keyed plug any more: polarity is checked by eye.** |
| Switch | Off-board, in the **switch tower** (12) on the side of the base: M6 toggle switch (6.2 mm hole), lever up. Sits between the battery and the regulator and cuts **the whole** system. 2-pin screw terminal `J_SWITCH1`, 20 AWG wire. |
| Main ESP32 | **ESP32-S3-DevKitC-1 N16R8** (16 MB flash, 8 MB octal PSRAM, 44 pins in 2 rows of 22). It may be a clone: verify it against Espressif's documentation. |
| Controller | Xbox controller over Bluetooth Low Energy (the ESP32-S3 has no Classic Bluetooth). The exact model and library are chosen in firmware. |
| Motors | 2 × yellow TT gear motors with 63 mm wheels, bolted to aluminium brackets. TB6612FNG VM comes after the switch; firmware limits PWM to 70 % maximum. |
| Driver | Commercial TB6612FNG breakout, plugged into the PCB through female headers. |
| 5 V regulator | Adjustable MP1584EN mini module, set to 5.0 V. |
| 3.3 V | From the 3V3 regulator on the main ESP32 board. |
| Sensors | 3 × VL53L0X (GY-530 module) at **3.3 V**, shared I2C bus, one independent XSHUT per sensor. |
| Proximity LEDs | **5 mm LEDs soldered directly on the PCB** (user decision), in the pairs of holes `J_LED_*`. Colors and threshold follow the university rules: FRONT red (top), LEFT yellow (middle), RIGHT green (bottom); on at ≤ 10 cm. |
| Power LED | **Green** 5 mm LED **soldered directly on the PCB** in `J_LED_POWER1` (user decision, replaces the off-board header + Dupont idea), with R1 = 1.5 kΩ. Not controlled by any GPIO. |
| ESP32-CAM | AI Thinker, no microSD, **removable** (programmed by removing it and using the user's ESP32-CAM-MB base), no programming header. |
| Camera | Standard OV2640 module (24-pin, 0.5 mm pitch, ~2 cm flex) in a printed cradle (16) on the camera arch (13), lens looking forward and tilted 25° down. See §7. |
| ESP32 ↔ ESP32-CAM UART | Direct traces on the PCB; no connector. |
| Home-made PCB rules | Tracks ≥ 0.4 mm (0.3 mm allowed where a track passes between pads). Holes ≥ 0.3 mm. **Pad diameter = drill diameter + 0.8 mm** on every pad hole, so copper is left on the pad after hand drilling. See §14. |
| Do NOT include | Battery monitoring, programming header, MOSFET protection, any other unrequested feature. |

---

## 4. Mechanical architecture

> For the schematic stage, use this section only to choose footprints and connectors that fit the final assembly.

### 4.1 Chassis (v8, from the user's visual guide `Guía visual Auxi v8`)

The chassis was printed in 3D after the first concept sketch. Facts taken from the guide (the user's own document; the printed parts live in the user's chassis project, not in this repository):

- **Size:** the robot is **144.1 mm long** (limit 150 mm). The lens of the camera sticks out only 0.15 mm beyond the front.
- **Weight:** estimated **762.5 g** with 3 ballast plates and the standard tray; target 800 g; **regulation limit 850 g**. Weigh the finished robot before adding more plates.
- **Printed parts:** 01 base · 02 tray (Warren truss, LITE or standard) · 03 columns ×4 · 04 sensor supports ×3 · 05 motor cradles ×2 · 07 V front or 08 curved front · 09 caster shims · 10 ballast box · 11 bottom lid · 12 switch tower · 13 camera arch · 14 battery door · 15 PCB spacers ×4 · 16 camera cradle (25°; 15° and 35° optional). Optional: 06 clamps, 17 printed motor bracket, 18 motor shims.
- **Base (01):** the floor of the robot. It has the pivots for the motor cradles and the front, the ballast opening and the caster columns.
- **Drive:** two TT motors bolted to aluminium brackets (2 × M3 × 30), fixed to the cradles 05 with 2 × M3 × 20 from below. **Wheels: 63 mm.**
- **Caster:** W420 (15.8 mm ball, 20.0 mm tall) on two base columns with 2 mm shims. The battery holder sits above it at Z = 24.
- **Battery:** two protected 18650 cells in a holder **at the rear**. The holder comes out at the back after removing two filament pins and the door (14). Charge outside the robot with a balancing 2S charger.
- **Ballast:** box 10 under the base, closed with lid 11 (2 screws). Three 37 mm steel strips of about 47 g each at the start.
- **Tray (02) and columns (03):** four columns carry the tray; the tray ties the columns together at the top and holds the sensor supports. Printed spacers (15) keep the PCB **10 mm above the tray**, so cables run under the PCB without being pressed against the solder joints.
- **Sensors:** three supports (04) at the front, FRONT / LEFT / RIGHT. Each GY-530 is bolted with 2 × M3 × 10 (nut behind). **Its wires are soldered to the module from behind (no connector at the sensor)**, leave through the window of the support wall and run backwards to the JST-XH connectors on the PCB. They never pass in front of the sensor or the camera.
- **Camera arch (13):** rests on the **two front corners of the PCB** and carries the camera cradle (16) and the team ID plate. Each leg takes an M3 × 25 from the top bar through the arch, the PCB and the spacer into the nut of column 03.
- **Switch tower (12):** hollow tower screwed to the base with 2 × M3, on the side of the robot. The M6 toggle switch sits inside with the lever up. The wires leave through a window on the inner side and enter `J_SWITCH1` directly (about 4 mm of bare wire).
- **PCB:** 110 × 140 mm, in its real KiCad position. The guide's 3D model is built from the heights of this `auxi.kicad_pcb`, so the 3D models of the parts must stay correct (see §21).

Coordinates for the PCB: origin at the center of the board, X to the right, Y forward (the guide uses the same convention).

**Cable routes (proposal in the guide).** The cables from the bottom level go up through the openings of the tray into the 10 mm gap, cross the PCB through pass-through holes, and reach their connector on top.

| Cable | From → to | Type |
|---|---|---|
| Battery | holder → `J_BAT1` (screw terminal), at the rear right of the PCB | 18–20 AWG, **check polarity** |
| Motors | rear of each motor → `J_MOTOR_*1` terminals on the right edge | 20–22 AWG |
| Switch | switch tower → `J_SWITCH1` | 20 AWG, never Dupont |
| Sensors | JST-XH connectors on the front edge → the three sensors (soldered from behind) | 25 cm or less, I2C at 100 kHz |
| Camera | ESP32-CAM → camera cradle on the arch | flex of about 2 cm in a gentle curve |

Forbidden: cables hanging over the wheels, and cables in front of the front sensor or the camera. Tie them with cable ties every 3–4 cm.

The guide also proposes three pass-through holes in the PCB (mm from the center of the board, X right, Y forward): sensors (X 38, Y 45, Ø 8 mm, 4.3 mm of free copper around), power for battery and motors (X 28, Y −48, Ø 10 mm, 5.0 mm free) and LEDs (X −41, Y −13, Ø 8 mm). **None of them was added: the PCB did not change (the board was already made).** The LED hole is not needed any more, because the 4 LEDs are soldered on the board. The other two are listed in §20.

### 4.2 Two levels (side view)

```
   camera arch (13)  ┌── single PCB (top level), 4 printed columns ──┐
                     │ ESP32-S3 · ESP32-CAM · TB6612 · MP1584 · LEDs │
   ──────────────────┴─────────┬── 10 mm gap (cables) ────┬──────────
                        tray (02) with the Warren truss (sensor supports)
   ═════ batteries (rear) · TT motors · ballast · wiring ═════════════
            base (01) · switch tower (12) on the side
```

**Why two levels:** the chassis floor is small. The motors take the sides, the ballast box and the battery take the middle and the rear, and the front piece takes the front. With two levels, the batteries and motors use the whole floor and the PCB uses the whole top area. It also has these benefits:

- The battery weight stays low, which makes the robot more stable.
- Cables run up short distances to their connectors.
- The USB ports, the ESP32-CAM and the connectors are reachable from above.

### 4.3 Mounting, height and space check

This check is mandatory before PCB design. It does not block the schematic.

- **Mounting holes:** the PCB has 4 M3 holes, one near each corner.
  - Each hole is 3.2 mm, with a keepout of about 6.5 mm diameter (no copper or parts), at least 4 mm from the edge.
  - The two front holes also carry the camera arch (13). **Chassis v8:** each hole takes one **M3 × 25** screw that goes through the arch (front) or the bar, the PCB and a printed spacer (15) into the nut of a column (03).
- **Height:** the PCB is **10 mm above the tray** (printed spacers), which keeps it above the motors, the battery holder and the cables.
  - In the v8 design the printed tray (plastic) sits between the solder joints of the PCB and the metal contacts of the battery holder, which is at the rear under the tray. Confirm the 5 mm clearance to metal on the real robot; if it is not met, add an insulating sheet.
- **Wheels:** the wheels are **63 mm** in diameter and sit outside the 110 mm PCB, at its sides (the guide shows no contact). Still check the real chassis.
- **Front and camera:** the PCB must not block the ball entry or the camera view. In v8 the camera arch rests on the two front corners of the PCB and the camera looks forward over the front edge.
- **Cable path:** see §4.1. The cables run up through the tray openings into the 10 mm gap and reach their connector (there is no enclosure). The pass-through holes proposed in the guide are **not** in the printed PCB (§20).
- **Size: 110 × 140 mm, set by the user.**
  - **Confirmed orientation (user photo):** the 110 mm side is the front of the robot and sits between the two wheels; the 140 mm side runs front to back. The robot is 144.1 mm long, so the 140 mm PCB almost spans the whole length. Check it against the front piece and the rear caster on the real chassis.
  - Check that all modules, connectors, M3 holes and antenna keepouts fit that outline, with room to route and to plug cables in by hand.
  - The ESP32-S3-DevKitC-1 is about 63–68 mm long. Its USB end is **1 mm** from the rear (bottom) edge (user request: at most 2 mm, as close as possible), so the USB-C plug overhangs the board edge and nothing blocks it. Keep the space around the USB end free of tall parts (the plug needs about 15 × 12 mm there).
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
| TT motor (body) | 70 × 22 × 18; ~37 wide including both shafts | Bottom level, in an aluminium bracket, plus the 63 mm wheel |
| VL53L0X GY-530 | ~25 × 11 | Off-board, on the printed supports (04) at the front |
| JST-XH 5-pin vertical | ~14.9 × 5.75 × 7 | One per sensor |
| 5 mm LED, THT | Ø5.0 (flange Ø5.8) × 8.6 high | The 4 LEDs soldered on the PCB (3D models in `auxi.3dshapes/`) |
| 2P screw terminal, 5.08 mm | ~10.2 × ~9.8 × ~10 | **4 on the board:** motors ×2, switch and battery |
| Battery pads (XT30UPB-M footprint) | 5.0 mm pitch, 1.8 mm holes, 3.0 mm pads | Hold the 5.08 mm screw terminal of the battery (0.08 mm pitch difference: the 1.8 mm holes absorb it) |
| 470 µF electrolytic | ~Ø10 × 13–20 | Height depends on voltage and brand |
| M3 hole | Ø3.2 with ~Ø6.5 keepout | 4 corners |

Modules plugged into female headers sit about 1 cm above the PCB. There is no enclosure, so this only matters for anything mounted above the PCB (check that nothing hits the modules).

---

## 5. Power

### 5.1 Architecture

```
BAT 2S ─ J_BAT (screw terminal) ─ J_SWITCH ═(wires)═ external switch ═(wires)═ J_SWITCH ─ +VBAT_SW
                                                                        │
+VBAT_SW ─┬─ R1 1.5 kΩ ─ green power LED (soldered on J_LED_POWER) ─ GND
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

- **Location:** the switch is **off the PCB, in the switch tower (12) on the side of the base**, with the lever up so it works without lifting the robot.
- **Electrical position:** it sits **between the battery and the regulator**. Per the rule in §5.1, it also cuts VM and all LEDs.
- **J_SWITCH on the PCB:** 2 terminals, `SW_BAT` (from the battery) and `SW_OUT` (to `+VBAT_SW`).
  - **All** of the robot's current flows through here, including motor start-up peaks.
  - Use a connector with a verified rating of ≥ 3 A. **Implemented:** a 2-pin, 5.08 mm pitch screw terminal (the same type as the motors and the battery), which needs no crimping.
  - **Do not use 2.54 mm headers with Dupont wires for this path.** Dupont jumpers handle about 1–2 A and their contact loosens.
- **Switch part:**
  - Continuous rating ≥ 3 A DC and ≥ 12 V DC, **verified in the datasheet**.
  - Estimate the system peak current and justify the choice.
  - **Chassis v8:** M6 threaded toggle switch (6.2 mm hole in the tower), 20 AWG wire. A slide switch or a mini rocker would not fit that hole.
  - State the minimum wire gauge.
- **J_LED_POWER on the PCB:** the 2-hole pad pair (`LED+` / `LED−`) where the **green** 5 mm LED is **soldered directly** (user decision: no header, no Dupont jumpers, no off-board LED). `LED+` is the right, square pad (long leg); `LED−` is the left pad (short leg, flat side). It is at the front right of the board, next to R1.
  - The current-limiting resistor is on the PCB. Size it for the whole 6.0–8.4 V range at about 2–5 mA.
  - **Implemented:** R1 = 1.5 kΩ. With a classic green LED (Vf ≈ 2.1 V) it gives about 2.6 mA at 6.0 V and 4.2 mA at 8.4 V; with a bright green (Vf ≈ 3 V) about 2.0 and 3.6 mA. Both are safe. (It was sized first for a blue LED.)
- **Silkscreen:** `SWITCH` / `ON-OFF` on J_SWITCH; `POWER LED` and polarity on J_LED_POWER. (Already printed on the board: it did not change.)

### 5.5 Battery

- No MOSFET protection is added. **The battery used to be a keyed XT30 plug; it is now a screw terminal, so nothing stops a reversed connection except the `BAT+` / `BAT−` silkscreen and the wire colors (red +, black −).** Check polarity with a multimeter before closing the switch. A reversed battery would feed the TB6612FNG and the MP1584 the wrong way round.
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
| XSHUT_RIGHT | GPIO4 | J1-4 | The XSHUT lines are the innermost of the north-going group: the first one goes to the east-most sensor connector (RIGHT), the last one to the west-most (LEFT), so no two lines cross |
| XSHUT_FRONT | GPIO5 | J1-5 | |
| XSHUT_LEFT | GPIO6 | J1-6 | |
| SCL | GPIO7 | J1-7 | SCL and SDA turn east above the XSHUT lines, at the height of pins 4 and 3 of the sensor connectors (SCL first, then SDA) |
| SDA | GPIO15 | J1-8 | |
| UART_CAM_TO_ESP | GPIO17 | J1-10 | ESP32 **RX**. The upper UART pin goes to the upper camera pin (IO14 = camera TX), the lower one to the lower camera pin, so the two lanes do not cross |
| UART_ESP_TO_CAM | GPIO18 | J1-11 | ESP32 **TX** (camera RX, IO13) |
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

- **Firmware note:** I2C is not on the Arduino default pins. Use `Wire.begin(15, 7)` (SDA = GPIO15, SCL = GPIO7). The camera UART is not on the default `Serial1` pins either: `Serial1.begin(115200, SERIAL_8N1, 17, 18)` (RX = GPIO17, TX = GPIO18). **The UART TX/RX swapped places compared with the previous mapping.**
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
- **Camera (chassis v8):**
  - The camera is a **standard OV2640 or OV3660 module** (24 pins, 0.5 mm pitch) with its normal **~2 cm flex**. The long flex of the first plan is **not needed any more**.
  - It sits in the printed cradle (16) under the camera arch (13): 9.2 mm pocket, lens through a 7.6 mm hole, tilted 25° down (15° and 35° cradles are optional). Fix it with a drop of hot glue or tape.
  - The ESP32-CAM is at the **front of the PCB, almost centered** (14.5 mm left of the centre, camera looking forward), with its FPC connector at the front end, under the cradle. The flex rises in a gentle curve.
  - Keep the flex path clear, with room to lift the connector latch by hand. Keep it away from the TB6612, the motor traces and the motor wires.
  - **To program the ESP32-CAM:** remove the two M3 × 25 screws of the arch legs and lift the arch (only ~3 mm are left above the front end of the module), then open the latch of the flex and take the module out.
  - Keep the field of view clear: no cable may pass in front of the camera.
  - If the video ever gets noise, lower XCLK in firmware (for example from 20 to 10 MHz).
  - **Matches / differs:** the guide says the ESP32-CAM is "at the front left". That is the same position (the board centre is 14.5 mm to the right of the ESP32-CAM centre).
  - During matches, the firmware should be able to switch the Wi-Fi video off: it can cut the Bluetooth link of the controller, and the rules forbid interfering with other teams.
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

- **Modules:** 3 GY-530 modules, bolted to the printed supports (04) at the front of the chassis (2 × M3 × 10, nut behind). **Their cables are soldered to the module from behind, with no connector at the sensor.** A connector on the sensor face would enter its field of view.
  - Verify the pins of the real module (typically VIN, GND, SCL, SDA, GPIO1, XSHUT).
  - GPIO1 is not used.
- **Power:** **3.3 V**, so the module's I2C pull-ups can never put 5 V on the ESP32.
- **Connectors:** one 5-pin JST-XH per sensor: `SENSOR_FRONT`, `SENSOR_LEFT`, `SENSOR_RIGHT`.
  - Same order on all three: 1 3V3, 2 GND, 3 SDA, 4 SCL, 5 XSHUT.
  - On the PCB all three sit **side by side along the front (top) edge, to the right of the camera**, LEFT (west), FRONT, RIGHT (east), 10 mm apart, with the pins running away from the edge (pin 1 = 3V3 nearest the edge). The 3V3, SDA and SCL buses are then three straight horizontal tracks that pass through the same-numbered pad of each connector, so they cannot cross. Each XSHUT track arrives at pin 5 from below.
- **I2C bus:** one shared bus (SDA on GPIO15, SCL on GPIO7, see §6.1), 100 kHz recommended, cables of 25 cm or less.
- **XSHUT:** RIGHT on GPIO4, FRONT on GPIO5, LEFT on GPIO6.
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
- **Expected behavior:** when the ball is inside the front piece, the FRONT sensor detects it and the red LED turns on. Document this.

---

## 10. Proximity LEDs (soldered on the PCB, university rules)

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
- **Location (user decision):** the 3 LEDs are **5 mm LEDs soldered directly on the PCB**, on the west side between the MP1584 and the ESP32-S3, top to bottom: FRONT (red), LEFT (yellow), RIGHT (green) (top view, front of the robot at the top). The robot has **no enclosure** (user decision), so the LEDs stay visible on the board, as the rules require, and no windows are needed.
- **Connection:** the footprint is still the 2-pin, 2.54 mm pad pair `J_LED_FRONT1` / `J_LED_LEFT1` / `J_LED_RIGHT1`. The LED legs go straight into those holes (no header, no Dupont jumpers).
  - The **right hole** (square pad, next to the resistor R3, R4 or R5) is the **anode (+, long leg)**. The **left hole** is the cathode (−, short leg, flat side of the LED).
  - Silkscreen with the name, color and polarity (`+` on the anode).
- **Resistors:** one per LED, on the PCB, sized from each color's Vf for about 2–5 mA at 3.3 V.
  - The current value is 470 Ω, which gives about 2.5–3 mA.
  - **R5 (green) is provisional:** it suits a classic green (Vf ≈ 2.1 V). A bright green (Vf ≈ 3 V) needs a recalculation.
- **Documentation note:** a LED soldered backwards simply does not light and is not damaged. Desolder it and flip it.
- **Power LED:** this is a different LED from these three. It is **green** (user), **soldered directly on the PCB** in `J_LED_POWER1`, at the front right of the board next to R1 (§5.4). It used to be an off-board LED next to the switch (a blue one in the first chassis guide): that is no longer true. Because the RIGHT proximity LED is also green, check that the two cannot be confused (they are on opposite sides of the board, and the power LED has the label `POWER LED`); if the rules require the indicator LEDs to be distinguishable, change the color of the power LED.
- **3D models:** the 4 LEDs have colored 5 mm models in `auxi.3dshapes/` (red, yellow, green; 8.6 mm high) so the 3D view and the chassis guide see the real height. The footprint is unchanged.

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
- **Motor connectors:** `MOTOR_LEFT` and `MOTOR_RIGHT` are 2-pin screw terminals (5.08 mm pitch) on the right (east) edge of the board. The cables come up from the motors through the tray and the 10 mm gap (§4.1).
  - Document that if a motor spins backwards, swap its wires or invert it in firmware.
- **Motor capacitor:** a 100 nF ceramic soldered across each motor's terminals, on the motor itself, not on the PCB.
- **Wheels:** the wheels are **63 mm** (they come with the TT motors in the v8 chassis). The motors are bolted to aluminium brackets and the brackets to the cradles 05.

---

## 12. External connection summary

Every off-board component connects to the PCB through a labeled connector. Nothing is soldered directly to the modules. **The only exception is the 4 LEDs (3 proximity + power), which are soldered directly into the `J_LED_*` and `J_LED_POWER1` holes (user decision).** The robot has **4 screw terminals** in total: 2 motors, the switch and the battery.

In the schematic, reference designators end in a digit (`J_BAT1`, `J_SENSOR_FRONT1`…) because KiCad requires it; otherwise it renumbers them at PCB stage.

| Reference | Off-board part | Connector type | Signals | Suggested location |
|---|---|---|---|---|
| J_BAT1 | 2S batteries | **2P screw terminal, 5.08 mm** (same as the motors), soldered into the pads of the XT30UPB-M footprint (5.0 mm pitch) | 1 BAT− (left pad, `-` mark), 2 BAT+ (right pad, `+` mark), matching the KiCad footprint's `−`/`+` marks | Rear edge, wire entry toward the rear (battery cables come from behind) |
| J_SWITCH1 | Switch | 2P screw terminal, 5.08 mm, ≥3 A | SW_BAT, SW_OUT | Near J_BAT1 |
| J_LED_POWER1 | Green power LED (5 mm, **soldered here**) | 2 holes, 2.54 mm | LED+ (right hole, square pad), LED− | Front right of the board, next to R1 |
| J_MOTOR_LEFT1 | Left motor | 2P screw terminal, 5.08 mm | M_L1, M_L2 | Right (east) edge of the board, next to the TB6612 (see §18: both motor terminals are on the same edge so the outputs of the driver do not have to cross the ESP32). The left motor cable is simply longer |
| J_MOTOR_RIGHT1 | Right motor | 2P screw terminal, 5.08 mm | M_R1, M_R2 | Right (east) edge |
| J_SENSOR_FRONT1 | Front sensor | JST-XH 5P vertical | 3V3, GND, SDA, SCL, XSHUT_FRONT | Front (top) edge, middle |
| J_SENSOR_LEFT1 | Left sensor | JST-XH 5P vertical | 3V3, GND, SDA, SCL, XSHUT_LEFT | Front (top) edge, west |
| J_SENSOR_RIGHT1 | Right sensor | JST-XH 5P vertical | 3V3, GND, SDA, SCL, XSHUT_RIGHT | Front (top) edge, east |
| J_LED_FRONT1 | Red LED (5 mm, **soldered here**) | 2 holes, 2.54 mm | LED+ (right hole, square pad), LED− | West side, top of the 3 |
| J_LED_LEFT1 | Yellow LED (5 mm, **soldered here**) | 2 holes, 2.54 mm | LED+ (right hole, square pad), LED− | West side, middle |
| J_LED_RIGHT1 | Green LED (5 mm, **soldered here**) | 2 holes, 2.54 mm | LED+ (right hole, square pad), LED− | West side, bottom |
| ESP32-CAM FPC | OV2640 camera | 24-pin flex (on the module itself), ~2 cm | Camera bus | Toward the front, under the camera cradle |

**Plug-in modules on the PCB:**

| Module | Connection |
|---|---|
| ESP32-S3-DevKitC-1 | 2 × 1×22 female headers (`U_ESP_J1`, `U_ESP_J3`) |
| ESP32-CAM | 2 × 1×8 female headers (`U_CAM1`, `U_CAM2`) |
| TB6612FNG | 2 × 1×8 female headers (`U_TB1`, `U_TB2`) |
| MP1584 | 4 pins soldered to the PCB (`U_MP1584`); verify against the real module |

**Connector rules:**

- **Power connectors** (battery, switch, motors): 5.08 mm screw terminals, rated for the real current. All four are the same part.
- **LEDs:** the 4 LEDs (3 proximity + power) are soldered directly in their holes. No header and no Dupont jumpers are used anywhere on the board.
- **Sensors:** polarized JST-XH, so 3V3 and GND cannot be swapped.
- **Placement:** all connectors on the edges, facing where their cable comes up, with room to plug and unplug by hand.

---

## 13. Capacitors

| Location | Value | Minimum voltage |
|---|---|---|
| `+VBAT_SW`, next to J_SWITCH | 470 µF low-ESR electrolytic + 100 nF X7R | 16 V (25 V preferred) |
| TB6612 VM, right at the header | 220 µF electrolytic + 100 nF | 16 V (C3 uses 220 µF 16 V: the rail is 8.4 V at most, user decision) |
| MP1584 input | 22 µF + 100 nF | 16 V (25 V preferred) |
| MP1584 output (`+5V`) | 100 µF electrolytic + 10 µF ceramic | 10 V |
| ESP32 5V pin (after the diode) | 47–100 µF + 100 nF | 10 V |
| ESP32-CAM 5V pin | 470 µF electrolytic + 10 µF + 100 nF | 10 V |
| `+3V3` next to TB6612 VCC | 10 µF + 100 nF | 10 V |
| Each sensor connector | 10 µF + 100 nF | 10 V |
| Each motor's terminals (on the motor) | 100 nF ceramic | 50 V |

- Use X7R or X5R ceramics in 0805 or larger, and account for capacitance loss with DC bias.
- Place each 100 nF as close as possible to the pin it decouples.
- Check that the electrolytics' height does not hit anything mounted above the PCB (there is no enclosure).

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
  - Power zone: battery terminal, J_SWITCH, TB6612 and motor terminals (east and rear-east of the robot); the MP1584 is on the west side next to its 5 V loads.
  - Logic and radio zone: ESP32-S3 (rear), ESP32-CAM and sensor connectors (front).
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

- The PCB is 110 × 140 mm with the 110 mm side at the front, between the wheels (user photo); it still has to be confirmed with the real chassis (see §20).
- The PCB sits 10 mm above the tray (printed spacers, M3 × 25 screws), above batteries, motors and cables, with the minimum clearance to metal contacts.
- The PCB does not hit the wheels or block the front piece or the camera.
- 4 M3 holes with keepouts.
- ESP32-S3 USB and buttons are reachable.
- ESP32-CAM is removable, with access to the FPC connector latch.
- The camera flex (~2 cm) path is clear and away from the power zone, and the arch can be lifted to take the ESP32-CAM out.
- All off-board connectors are reachable and face where their cables come up: battery, switch, power LED, motors, sensors and LEDs.
- The chassis has a planned cable path (§4.1). The pass-through holes of the guide are still undecided (§20).
- No collisions between components, modules, cables and the chassis.

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

**Board orientation (given by the user, photo of the chassis):**
- Board outline: **x 106.07–216.07 mm, y 47.5–187.5 mm** in KiCad coordinates (110 × 140 mm, 2 mm corner radius).
- The **110 mm side is the robot FRONT** (it sits between the wheels) and the 140 mm side runs front to back. In KiCad: **top edge (y = 47.5) = FRONT**, **bottom edge (y = 187.5) = REAR**, left edge (x = 106.07) = robot LEFT side, right edge (x = 216.07) = robot RIGHT side.
- The ESP32-CAM is at the front with the camera looking forward (almost centered, see below); the ESP32-S3 is at the rear with its USB port at the rear edge (user request: at most 2 mm from the edge).

**History.** (1) 90 × 100 mm, 2 layers, 75 vias. (2) 110 × 140 mm single layer with the 110 mm side assumed to run front to back, the ESP32-S3 26 mm inside the edge. (3) After the user's photo and the "USB at the edge" request, the board was **re-placed and re-routed from scratch** in the final orientation, with the camera in the front-left corner and the sensors in the middle of the front edge. (4) At the user's request the camera and the sensors were swapped: the camera is now **almost centered** at the front and the sensors are to its right (this layout). Earlier versions are in the Git history.

**Placement (x, y of pin 1 in mm):**

| Part | Position | Notes |
|---|---|---|
| ESP32-S3 | `U_ESP_J1` (149.64, 125.90), `U_ESP_J3` (172.50, 125.90) | Pins run toward the rear edge. **The module end (USB side) is 1.0 mm from the rear edge** (module end at y ≈ 186.5, edge at 187.5), so the USB-C plug overhangs the board. Antenna keepout `ANT_ESP32` x 148.1–174.1, y 116.2–125.2 |
| ESP32-CAM | `U_CAM1` (135.07, 86.30), `U_CAM2` (157.93, 86.30), rotated 180° | At the front, centre at x = 146.5 (14.5 mm left of the board centre, 161.07). Pins at the rear end of the module, antenna and flex at the front edge. Keepout `ANT_CAM` x 132.5–160.5, y 48–60.5. Pin 1 (+5V) is the south-most pin of the west row; the UART pins are pins 4 and 6 of the same row |
| TB6612FNG | `U_TB1` (178.64, 148.76), `U_TB2` (196.42, 148.76), rotated 180° | Right next to `J3`. Control row toward the ESP32, power row toward the east edge |
| MP1584 | centre (118.0, 160.0), rotated 90° | West side. OUT+ toward the camera (north), IN+ toward the rear (south) |
| Sensors | `J_SENSOR_LEFT1` (165.0, 53.0), `FRONT1` (175.0, 53.0), `RIGHT1` (185.0, 53.0), rotated 270° | Along the front edge, to the right of the camera, pins running south |
| Proximity LEDs | pad pairs at x = 133.5, y = 154.0 / 159.5 / 165.0 (FRONT red, LEFT yellow, RIGHT green), resistors R3–R5 at x = 138–146 | West side, between the MP1584 and the ESP32-S3. The 5 mm LEDs are soldered directly into the pads; `+` is the right (square) pad |
| Motors | `J_MOTOR_RIGHT1` (209.5, 115.76), `J_MOTOR_LEFT1` (209.5, 159.76) | East edge, wire entry toward the edge |
| Power | `J_SWITCH1` (209.5, 176.0), `J_BAT1` (185.64, 182.5), `J_LED_POWER1` (204.0, 53.0) | The battery terminal and the switch terminal are the same 5.08 mm part. The big VBAT capacitors C1, C3, C5 and the small ones form a compact cluster at x ≈ 192–204, y ≈ 80–135 (not a long column: see below) |
| M3 holes | 5 mm from each corner | |

- **Why this arrangement (single-layer planarity).** Every signal must reach its destination without crossing another one, so the ESP32 header pins are in the same order as the destinations around the module:
  - **West column `J1`, top to bottom:** 3V3, the 3 XSHUT, SCL, SDA (north-going, 1 mm apart, **between the camera and the ESP32**, then east above the camera to the sensors along the front edge), the 2 UART lines (north, inside the camera columns), the 3 LED lines (south-west, to the LED headers) and the 5V pin (south).
  - **East column `J3`:** the 7 TB6612 signals, straight horizontal tracks to the driver.
  - The camera 5V track runs from the MP1584 north along the west side to y = 140, then east to x = 138 and north into the camera, **outside** (west of) the two UART lanes (x = 140.5 and 142.5). Both enter the camera from its inside; the UART lanes turn west to pins 4 and 6, and the 5V lane to pin 1.
  - The 3V3 for the TB6612 VCC pin crosses the ESP32 socket **under the module** and passes between pads 9 and 10 of `J3` at 0.3 mm; the same net then runs north at x = 190.5 and west along the top edge to pin 1 of the sensor connectors. This is why the big capacitors and `J_LED_POWER1` are kept in a compact cluster at x ≥ 192 and not along a long column: a long +VBAT_SW column on the east side would wall in the VCC pin.
  - The bottom strip under the ESP32 (about 7 mm) carries the +VBAT_SW track (1.2 mm) from the battery terminal to the MP1584; D1, C9 and C10 sit west of `J1` next to the 5V pin.
  - **Why the camera is not exactly centered:** a camera at the very center needs its 5V and UART tracks to reach it from the south, and the sensor tracks (that run north between the camera and the ESP32) would have to cross them. On one layer that is impossible without a jumper. The camera is therefore moved 14.5 mm to the left of the centre, which leaves the space between the camera and the ESP32 free for the sensor tracks.
- C11 (camera 470 µF) sits west of the camera lanes, in the corridor below the camera.
- Silkscreen: module outlines for the ESP32-S3 (antenna end line), ESP32-CAM and MP1584 (its own footprint), TB6612 outline on `F.Fab`, and the labels of §15 (`^ FRONT` at the top, `REAR (USB)` at the bottom). Reference designators stay on `F.Fab`.

**Routing method.** The MCP KiCad tools were not available, so the work was done with scripts (KiCad 10 Python API) and **Freerouting 2.1.0**:
1. The placement was generated by a script (fixed module positions plus a greedy search for the passive parts).
2. The camera UART lanes, the camera 5V lane, the five sensor lines, the sensor buses and the long 3V3 path were laid out **by hand** (fixed wires) because the autorouter could not find them on its own; Freerouting routed everything else **on one layer only**, with GND routed as a normal net, 0.2 mm clearance and 0.3 mm base track width.
3. The GND tracks that were not needed were removed one by one, as long as the pour still reached every GND pad (down to 4 short GND segments).
4. Every track that could be widened to 0.4 mm without breaking the 0.2 mm clearance was widened.
- The scripts are not part of the repository. Re-running the layout means repeating this procedure in KiCad (or with Freerouting).

**Routing result (final board):**

| Item | Value |
|---|---|
| Tracks | 167, all on `B.Cu`, 0 vias, 0 jumper wires |
| Widths | 0.3 mm × 5 (37 mm), 0.4 mm × 64, 0.6 mm × 44, 0.8 mm × 4 (GND), 1.2 mm × 50 |
| Net classes | `PWR_BAT` 1.2 mm (`+VBAT_SW`, `SW_BAT`, `M_*`), `PWR_LOW` 0.6 mm (`+5V`, `+5V_ESP`), `GND` 0.8 mm, `Default` 0.4 mm |
| Pour | `GND_PLANE` on `B.Cu`, one zone, thermal reliefs, no isolated islands |
| Drills | 0.8, 0.95, 1.0, 1.1, 1.3, 1.8 mm (pad holes) and 4 × 3.2 mm (M3) |
| DRC (`kicad-cli pcb drc`, zones refilled, custom rules) | **0 errors, 0 unconnected items**. One silkscreen warning remains (the outline of the ESP32 socket touches the "K" mark of D1) |
| ERC | 0 errors, 0 warnings |
| Schematic parity | The PCB nets match the schematic pin by pin (173 pads, 0 mismatches); KiCad's parity report only shows the net-name prefix and field warnings |

**Things to review by eye before etching (DRC cannot judge them):**
- The `+3V3` track to the TB6612 VCC pin passes **between two pads of the ESP32 header `J3`** at 0.3 mm (and runs under the module): check it for breaks when etching. The +VBAT_SW track (1.2 mm) runs along the rear strip, close to the board edge (0.5 mm clearance).
- The 0.3 mm tracks (I2C, UART, LED signals) run in tight bundles with 0.2 mm gaps. They are the most likely to break or short when etching. Print the copper page, look at it under light, and consider a slightly longer exposure.
- I2C/UART tracks versus the motor tracks and the antenna keepouts (§14). The motor tracks are all on the east side of the board.
- Any 0.3 mm track that passes **between header pads** (the pads are 1.8 mm wide on a 2.54 mm pitch, so each gap is 0.74 mm and a 0.3 mm track keeps 0.22 mm of clearance on each side). Look at the ESP32-S3 and TB6612 header rows.
- Capacitor heights (§14), the free space around the ESP32 USB end, and that the chassis lets the USB cable come out at the rear edge.

**Pending:** the 2D mechanical check with real measurements (§4.3/§17), and the open items in §20.

### Stage 3: chassis v8 and wiring update (files updated; the PCB itself did NOT change)

The chassis was printed in 3D (guide `Guía visual Auxi v8`, see §4.1) and three things changed in how the PCB is used. **The board that was etched is the same one: copper, footprints, positions and tracks are untouched.** Only the 3D models, the part descriptions and the documents changed.

1. **All 4 LEDs are soldered directly on the PCB**: the 3 proximity LEDs (already so) and now also the **green power LED** (`J_LED_POWER1`, R1 = 1.5 kΩ). There is no header and no Dupont jumper for any LED.
2. **The same 5.08 mm screw terminal is used for the switch and for the battery**, so the robot has **4 screw terminals** (motor left, motor right, switch, battery). The battery terminal is soldered into the pads of `J_BAT1`, which keep the XT30UPB-M footprint (5.0 mm pitch, 1.8 mm holes, 3.0 mm pads): the 5.08 mm terminal pins fit in the 1.8 mm holes with 0.08 mm of difference.
3. **Chassis v8** (§4): 144.1 mm long, 63 mm wheels, W420 caster, battery at the rear, ballast box, switch tower, camera arch with a standard ~2 cm camera flex, PCB 10 mm above the tray on printed columns with M3 × 25 screws.

What was changed in the KiCad files:

| File | Change |
|---|---|
| `auxi.kicad_pcb` | **3D models only.** `J_LED_FRONT1` / `J_LED_LEFT1` / `J_LED_RIGHT1` / `J_LED_POWER1`: the pin-header model was replaced by a colored 5 mm LED (`auxi.3dshapes/LED_D5.0mm_Red.wrl`, `_Yellow.wrl`, `_Green.wrl`; offset `0 −1.27 0` so the LED sits between the two pads; 8.6 mm high). `J_BAT1`: the XT30 model was replaced by the Phoenix MKDS 5.08 mm terminal model (offset `−0.04 0 0` to split the 0.08 mm pitch difference), and its value is now `BAT 2S (bornera)`. |
| `auxi.kicad_sch`, `auxi.kicad_sym` | **No electrical change** (same nets and pins). The description of `Conn_LED` now says the LED is soldered in the holes, `J_BAT1` value is `BAT 2S (bornera)` and the notes of the power block say the same. |
| `auxi.3dshapes/` | New: the three LED models (VRML, 0.1 inch units, as KiCad expects). |
| `docs/` | **Pending regeneration** (needs `kicad-cli`, which was not available when these edits were made): schematic PDF, 1:1 PDF and the three renders must be regenerated, and ERC/DRC re-run. Only the BOM row of `J_BAT1` was edited by hand. |

**Limitation:** the library copy `auxi.pretty/PinHeader_1x02_P2.54mm_Vertical.kicad_mod` still points to the pin-header 3D model, because its pads are the right ones for the LEDs and the footprint was not allowed to change. The LED models live in the PCB file. If someone resets those 4 footprints from the library ("Update footprint from library"), the header model comes back and the LED model has to be set again.


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
| 1 | Switch ≥ 5 A DC and ≥ 12 V DC, **verified in its datasheet** (**M6 threaded toggle switch**, 6.2 mm hole in the switch tower). Confirm it against the measured stall current. |
| 1 | 5 mm **green** LED (power, **soldered on the PCB** at `J_LED_POWER1`, with R1 = 1.5 kΩ) |
| 1 + 1 + 1 | 5 mm LEDs: red, yellow and green (proximity), **soldered directly on the PCB** (see the R5 note in §10) |
| 3 | VL53L0X GY-530 modules |
| 3 | 5-pin JST-XH cables, ≤ 25 cm (with XHP-5 female housing) |
| 1 | Standard OV2640 or OV3660 camera module (24 pins, 0.5 mm pitch, ~2 cm flex), glued in the cradle (16) |
| 2 | TT gear motors + 2 × 63 mm wheels + 2 aluminium TT brackets + 1 W420 caster |
| 2 | 100 nF 50 V ceramic capacitors for the motor terminals |
| 1 | 2×18650 battery holder (with bare wire leads, 18–20 AWG, to the screw terminal) + 2 protected 18650 cells. No XT30 plug is needed |
| — | 20 AWG wire (battery and switch) and 22 AWG wire (motors) |
| 4 | M3 × 25 screws for the columns (printed columns 03 and spacers 15 replace the standoffs). The full list of printed parts, screws and nuts is in the chassis guide v8 |

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

7. ~~Does the 150 × 150 mm chassis include the wheels?~~ **Resolved by the v8 guide:** the robot is 144.1 mm long (limit 150 mm) with 63 mm wheels at the sides of the 110 mm board. The width is not stated in the guide: still to confirm against the real robot and the rules.
8. ~~Orientation of the 110 × 140 mm PCB~~ **Resolved by the user's photo:** the 110 mm side is the front (between the wheels). The robot is 144.1 mm long, so the 140 mm board nearly spans it: still to check on the real chassis that it clears the front piece and the rear caster, that the USB cable at the rear edge has a way out, and that the arch sits on the two front corners as drawn.
9. Real measurements of the TT motor and the battery holder, and the height available on the bottom level. (Wheels: 63 mm, from the v8 guide.)
10. ~~Location of the proximity LEDs~~ **Decided by the user:** soldered directly on the PCB (west side). There is no enclosure, so the LEDs are visible. The **power LED is now also soldered on the PCB**, at the front right: still to check only that the green power LED is not confused with the green proximity LED.
11. ~~Camera flex length~~ **Resolved by the v8 guide:** a standard module with its ~2 cm flex, in the cradle on the arch.
12. Height of the electrolytic capacitors that will be bought (C1, C3, C11), and a low-profile 6.3 × 7 mm part for C9 (it sits next to the ESP32 USB plug area).
13. **Pass-through holes in the PCB** (guide proposal, §4.1): sensors (X 38, Y 45, Ø 8 mm) and power (X 28, Y −48, Ø 10 mm). They are **not** in the printed board. Decide whether the cables go around the board edge or whether those two holes are drilled by hand (and added to KiCad, then DRC). The LED hole of the guide is no longer needed.
14. **Reverse battery connection:** the battery is now a screw terminal, so it has no mechanical keying. Confirm the polarity marking on the real board (`-` on the left pad, `+` on the right pad) and mark the wires.
15. **Terminal pitch:** the battery terminal has 5.08 mm pins in 5.0 mm pads. Confirm it fits without forcing it (the 1.8 mm holes should absorb the 0.08 mm). Also confirm which side its wire entry faces (the 3D model assumes the rear edge).
16. **Weight:** 762.5 g estimated, 850 g is the limit. Weigh the real robot.
17. The v8 guide still shows the **old wiring** for the LEDs (Dupont cables to LEDs on the supports and a blue power LED in the tower) and for the battery (XT30). It needs to be regenerated with the changes of Stage 3.

---

## 21. Repository files

| Path | Notes |
|---|---|
| `auxi.kicad_pro` / `auxi.kicad_sch` / `auxi.kicad_pcb` | Project, schematic, PCB (single layer, 110 × 140 mm, routed) |
| `auxi.kicad_dru` | Custom DRC rules: pad ring ≥ 0.4 mm, hole ≥ 0.3 mm |
| `auxi.kicad_sym`, `auxi.pretty/` | Project libraries, loaded through `sym-lib-table` and `fp-lib-table` with `${KIPRJMOD}` |
| `auxi.3dshapes/` | 3D models of the 5 mm LEDs (red, yellow, green; VRML), used by the PCB through `${KIPRJMOD}`. Added in Stage 3 |
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
