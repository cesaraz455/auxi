# auxi — Soccer Robot Controller Board

**auxi** is a small soccer robot: a first-semester university project. This repository holds the KiCad design for its single custom PCB.

> **Status: Stage 2 (PCB) — single-layer layout finished.** The schematic passes ERC with no errors or warnings, and the PCB passes DRC with **0 violations and 0 unconnected items**. The design is still **preliminary** until the purchased modules are measured (see [Open items](#open-items)).

---

## The robot

- **Drive:** 2 yellow TT gear motors plus 1 rear caster wheel.
- **Chassis:** 3D-printed, about 150 × 150 mm. The front has a built-in **concave area that guides the ball**.
- **Two-level layout:**
  - **Bottom level (chassis):** batteries, motors, caster and wiring.
  - **Top level:** one **110 × 140 mm PCB** on four M3 standoffs, sitting above the batteries and motors.
- **Main MCU:** ESP32-S3-DevKitC-1 N16R8. It reads the sensors, drives the motors and LEDs, and receives commands from an **Xbox controller over Bluetooth Low Energy**. The ESP32-S3 has no Classic Bluetooth.
- **Camera:** AI Thinker ESP32-CAM. It streams live video over Wi-Fi to a phone and shows the distances it receives from the main ESP32 over UART. The OV2640 camera is mounted at the front of the chassis on a long flex cable.
- **Sensors:** 3 × VL53L0X time-of-flight sensors (GY-530 modules) in the concave front area: FRONT (0°), LEFT (+45°) and RIGHT (−45°).
- **Proximity LEDs:** 3 LEDs whose colors are set by the university rules. Each LED turns on when its sensor reads **≤ 100 mm**.

## What is on the PCB

| On the PCB (plug-in modules and parts) | Off the PCB (connects through a connector) |
|---|---|
| ESP32-S3-DevKitC-1 (2 × 1×22 female headers) | 2S 18650 battery (XT30) |
| ESP32-CAM (2 × 1×8 female headers) | Power switch (screw terminal) |
| TB6612FNG breakout (2 × 1×8 female headers) | Green power LED (2-pin header) |
| MP1584EN mini buck module (5.0 V) | 2 TT motors (screw terminals) |
| Capacitors, resistors, Schottky diode | 3 VL53L0X sensors (JST-XH 5P) |
| **3 proximity LEDs (5 mm, soldered directly on the board)** | |
| All connectors | |
| | OV2640 camera (flex cable into the ESP32-CAM) |

## The board: one copper layer, soldered from below

The board is made for **home etching and hand drilling**, because most local fabs do not plate the holes.

- **One copper layer: `B.Cu` (the bottom face).** Every track is on the bottom. The rest of the bottom face is a **GND pour** (copper plane) that replaces the ground tracks.
- **All parts are through-hole (THT).** They go on the **top face** (the face without copper). Their legs go through the holes and are **soldered from below**, on the copper side. No hole needs plating, because there is nothing to connect on the top face.
- **No vias and no jumper wires.** The 167 tracks are all on `B.Cu`, and no track crosses another one.
- **Size: 110 × 140 mm**, 4 × M3 holes 5 mm from each corner, 2 mm rounded corners.
- **Orientation (given by the user, see the photo of the chassis):** the **110 mm side is the FRONT of the robot** (it sits between the two wheels) and the 140 mm side runs front to back. In KiCad the **top edge is the robot FRONT**, the bottom edge is the REAR, the left edge is the robot LEFT side and the right edge is the RIGHT side. The ESP32-CAM is at the front (camera looking forward) and the ESP32-S3 is at the rear.
- The `F.Cu` layer has no copper at all. A keepout covers the whole top face so no track or pour can appear there by mistake. Only the pads show on top.

![Top view](docs/auxi_pcb_top.png)

![Bottom view (copper side)](docs/auxi_pcb_bottom.png)

### Layout in one paragraph

Seen in KiCad (front of the robot = top edge):

- **Front (top):** the **ESP32-CAM** is at the front, almost centered (its centre is 14.5 mm to the left of the board centre), with its antenna and flex connector at the top edge, so the camera looks forward. The three **sensor connectors** are side by side along the top edge, to the right of the camera (LEFT, FRONT, RIGHT from left to right). A perfectly centered camera is not possible on one copper layer, because the 5 V and UART tracks of the camera would have to cross the sensor tracks.
- **Rear (bottom):** the **ESP32-S3** sits in the middle of the rear edge with its **USB end only 1 mm from the bottom edge** (user request: at most 2 mm). The USB-C plug overhangs the edge, so the cable plugs in without any obstacle.
- **East side:** the TB6612FNG is right next to the ESP32-S3 header row `J3` (short control signals), with the motor terminals on the right edge and the big capacitors, the power LED header and the battery/switch connectors on the east side and the bottom-right corner.
- **West side:** the MP1584, the 3 proximity LEDs (soldered directly on the board) and the big capacitor of the camera.
- The free areas between the groups are GND plane.

## Why the pins moved

A single layer cannot cross two tracks. To avoid any crossing (and any jumper wire), the GPIOs were **reassigned** so the signals leave the ESP32 headers in the same order as their destinations. The firmware has not been written yet, so this costs nothing. See the new table in [Pin map](#pin-map-gpio).

The mapping changed **once more** when the board orientation was fixed (110 mm side = front): the I2C pins and the camera UART pins moved to the order that the new layout needs, and the UART **TX and RX swapped places** (`GPIO18` is now the ESP32 transmit pin). Always check the table below.

## Power architecture

```
BAT 2S (6.0–8.4 V) ─ XT30 ─ external switch ─ +VBAT_SW
+VBAT_SW ─┬─ R1 ─ green power LED
          ├─ bulk caps ─ TB6612 VM (motors)
          └─ MP1584 ─ +5V ─┬─ D1 Schottky ─ +5V_ESP ─ ESP32-S3 5V pin
                           └─ ESP32-CAM 5V pin
ESP32-S3 3V3 pin ─ +3V3 ─ VL53L0X sensors, TB6612 logic (I2C pull-ups are on the GY-530 modules)
```

- Nothing connects to the battery before the switch. When the switch is off, the whole robot is off.
- D1 keeps the USB port and the robot from powering each other while you program the board.
- The firmware **must** limit motor PWM to **70 %**, which gives about 5.9 V average at 8.4 V.

## Pin map (GPIO)

**Left header row (`U_ESP_J1`)**

| Signal | GPIO | Header pin |
|---|---|---|
| XSHUT_RIGHT | GPIO4 | J1-4 |
| XSHUT_FRONT | GPIO5 | J1-5 |
| XSHUT_LEFT | GPIO6 | J1-6 |
| SCL | GPIO7 | J1-7 |
| SDA | GPIO15 | J1-8 |
| UART_CAM_TO_ESP (ESP32 **RX**) | GPIO17 | J1-10 |
| UART_ESP_TO_CAM (ESP32 **TX**) | GPIO18 | J1-11 |
| LED_FRONT | GPIO8 | J1-12 |
| LED_LEFT | GPIO9 | J1-15 |
| LED_RIGHT | GPIO10 | J1-16 |

**Right header row (`U_ESP_J3`)**

| Signal | GPIO | Header pin |
|---|---|---|
| PWMB | GPIO1 | J3-4 |
| BIN2 | GPIO2 | J3-5 |
| BIN1 | GPIO42 | J3-6 |
| STBY | GPIO41 | J3-7 |
| AIN1 | GPIO40 | J3-8 |
| AIN2 | GPIO39 | J3-9 |
| PWMA | GPIO21 | J3-18 |

- I2C is **not** on the Arduino default pins any more: call `Wire.begin(15, 7)` (SDA = GPIO15, SCL = GPIO7).
- The camera UART is also not on the default `Serial1` pins: call `Serial1.begin(115200, SERIAL_8N1, 17, 18)` (RX = GPIO17, TX = GPIO18).
- Spare pins: GPIO11, 12, 13, 14 and 16.
- Still avoided on purpose: strapping pins (0, 3, 45, 46), USB (19, 20), console (43, 44), octal PSRAM (35, 36, 37), RGB LED (38, 48) and 47.
- R2 (10 kΩ to GND) still holds `STBY` low while the ESP32 boots, so the motors stay off.

## Proximity LEDs (university rules)

| Sensor | LED color | Position on the board (top view, front at the top) | Pads |
|---|---|---|---|
| FRONT | Red | top | `J_LED_FRONT1` |
| LEFT | Yellow | middle | `J_LED_LEFT1` |
| RIGHT | Green | bottom | `J_LED_RIGHT1` |

A LED is **on** when its sensor reads ≤ 100 mm. It is **off** when the reading is above 100 mm, invalid or out of range. The firmware may average readings, but it must not use hysteresis that keeps a LED on above 100 mm.

**Decision (user): the three proximity LEDs are 5 mm LEDs soldered directly on the board**, in the pairs of holes marked `J_LED_FRONT1`, `J_LED_LEFT1` and `J_LED_RIGHT1` (no header and no Dupont jumpers). They are stacked on the west side, between the MP1584 and the ESP32-S3, top to bottom: FRONT (red), LEFT (yellow), RIGHT (green). In each pair of holes the **right hole** (the **square pad**, next to the resistor R3, R4 or R5) is the LED `+` (anode, the **long leg**) and the **left hole** is the `−` (cathode, the short leg and the flat side of the LED). A LED soldered backwards does not light and is not damaged. The robot has **no enclosure** (user decision): the board stays visible on top of the chassis, so the LEDs are visible as the university rules require and no windows or holes are needed. Solder them at a normal height.

The **power LED** is different: it is a **green** 5 mm LED that stays off the board, connected to the 2-pin header `J_LED_POWER1` (top right) with Dupont jumpers. Resistor `R1` (1.5 kΩ) gives it about 2.6 mA at 6.0 V and 4.2 mA at 8.4 V with a classic green LED (2–3.6 mA with a bright one). Because the RIGHT proximity LED is also green, check that the two cannot be confused (they are far apart on the board, but both are green).

## Manufacturing rules for the home-made board

These rules were requested so the tracks survive etching and the pads keep copper after drilling by hand.

| Item | Rule | How it is enforced |
|---|---|---|
| Track width | **≥ 0.4 mm**, except **0.3 mm** where a track passes between pads or through a tight bundle | Net classes in `auxi.kicad_pro` and `auxi.kicad_dru`. KiCad refuses anything under 0.3 mm |
| Power tracks | `+5V`, `+5V_ESP`: 0.6 mm. Battery, switch, VM and motors: 1.2 mm | Net classes |
| Clearance | 0.2 mm between copper objects | DRC |
| Smallest hole | **0.3 mm** (the real smallest drill is 0.8 mm) | DRC rule `min_hole_0.3mm` |
| Pad size | **pad = drill + 0.8 mm** on every pad hole (0.4 mm of copper all around) | DRC rule `pad_ring_0.4mm` |
| Via | none are used. The default via is 1.1 mm pad / 0.3 mm hole (same 0.8 mm rule) | `auxi.kicad_pro` |
| Edge | 0.5 mm of copper clearance to the board edge | DRC |

Result in the final board: 167 tracks, of which **162 are 0.4 mm or wider** and **5 are 0.3 mm** (37 mm in total). The 0.3 mm tracks are the ones that squeeze between pad rows or run in tight bundles (I2C, UART and LED signals); the router could not widen them without breaking the 0.2 mm clearance.

### Drill table

| Drill (mm) | Pad (mm) | Holes | Used by |
|---|---|---|---|
| 0.80 | 1.6 | 48 | Resistors and capacitors up to 8 mm |
| 0.95 | 1.75 × 1.95 | 15 | JST-XH sensor connectors |
| 1.00 | 1.8 (2.0 for the two 470 µF capacitors) | 96 | Female headers (ESP32-S3, ESP32-CAM, TB6612FNG), 2-pin header and LED holes, MP1584 pins, 470 µF capacitors |
| 1.10 | 2.2 | 2 | 1N5819 diode |
| 1.30 | 2.6 | 6 | 5.08 mm screw terminals |
| 1.80 | 3.0 | 2 | XT30 connector |
| 3.20 | no pad | 4 | M3 mounting holes (non-plated) |

Header pins are 0.64 mm square (0.9 mm diagonal), so a 1.0 mm hole is enough. Check the legs of your own parts before drilling: if a part is thicker than the hole, open that hole with the next bit size (the pad still keeps at least 0.3 mm of ring).

## Building the board

1. Print the **B.Cu page** of [`docs/auxi_pcb_real_scale.pdf`](docs/auxi_pcb_real_scale.pdf) at **100 % scale** (no "fit to page"). The 100 mm ruler under the board must measure 100 mm.
   - **Page 2** is the copper **as seen from above** (not mirrored). For toner transfer, press the printed face against the copper. The result reads correctly from the copper side.
   - **Page 3** is the same copper **mirrored**, for methods where the sheet lies printed face up.
   - Black is copper. The small white dots in the pads are drill guides.
2. Etch, clean and drill (see the drill table). Drill from the copper side so the guide dots center the bit.
3. Mount the components on the **top face** and solder them **from below**. Order: low parts first (resistors, diode, ceramic capacitors), then electrolytic capacitors, then sockets and connectors.
4. Page 1 of the PDF shows where every part goes (top view with silkscreen and references).
5. Solder the 4 pins of the MP1584 **last**, after you set its output (see the warnings below).

The **USB end** of the ESP32-S3 is only **1 mm** from the rear board edge, so the plug overhangs the edge: keep the area around it free of tall parts and make sure nothing on the chassis blocks the cable at the rear (the plug and cable need about 15 × 12 mm). Use a low-profile **6.3 × 7 mm** capacitor for C9 (100 µF). The long `+3V3` and `+VBAT_SW` tracks run along the east side and the bottom strip of the board, away from the I2C and UART tracks.

## Repository layout

| Path | Contents |
|---|---|
| `auxi.kicad_pro` | KiCad project, including the net classes and the design rules |
| `auxi.kicad_dru` | Custom DRC rules (pad ring and hole size) |
| `auxi.kicad_sch` | Schematic |
| `auxi.kicad_pcb` | Single-layer PCB (110 × 140 mm) |
| `auxi.kicad_sym` | Project symbol library (module header rows, etc.) |
| `auxi.pretty/` | Project footprint library. Four standard footprints were copied here with bigger pads (pad = drill + 0.8 mm). `MP1584_Mini_PROVISIONAL` has estimated pads |
| `sym-lib-table`, `fp-lib-table` | Project library tables |
| `auxi_drc_violations.json` | Last DRC report (empty list of violations) |
| `docs/auxi_esquematico.pdf` | Schematic PDF export |
| `docs/auxi_pcb_real_scale.pdf` | 1:1 prints: top view, copper, mirrored copper |
| `docs/auxi_bom_pcb.csv` | PCB BOM |
| `docs/auxi_pcb_top.png`, `docs/auxi_pcb_bottom.png`, `docs/auxi_pcb_placement.png` | Renders of the board |

The full design specification (confirmed decisions, power, pinout, connectors, capacitors, PCB rules and open items) is in [`CLAUDE.md`](CLAUDE.md). It is the source of truth for every design decision.

## Opening the project

1. Install **KiCad 10**. The files use the KiCad 10 file format, so older versions cannot open them.
2. Open `auxi.kicad_pro`. The project libraries (`auxi`) load through the local `sym-lib-table` and `fp-lib-table` using `${KIPRJMOD}`.
3. The schematic points to the four custom `auxi:` connector footprints (bigger pads), so "Update PCB from Schematic" keeps them. Do not swap them back to the standard KiCad ones.

## Assembly warnings

1. **Set the MP1584 to 5.0 V with a multimeter BEFORE plugging in any module.**
2. **Check battery polarity** (`BAT+` / `BAT−`) before connecting it.
3. **The switch path carries all of the robot's current.** Use 20 AWG wire and the screw terminal. Never use Dupont jumpers there.
4. **Remove the ESP32-CAM to program it**, using the ESP32-CAM-MB programmer base.
5. **Motor PWM must never exceed 70 %.**
6. **Use protected 18650 cells**, or charge them outside the robot with a balancing 2S charger. The PCB has no BMS or charging circuit.
7. **Check every track under a magnifier** before soldering: with 0.2 mm gaps, a small etching defect can short two tracks. Check the 0.3 mm tracks for breaks as well.

## Roadmap

- [x] **Stage 1:** schematic, footprints and ERC
- [x] **Stage 2:** 110 × 140 mm single-layer PCB layout, DRC 0 violations
- [ ] Close the open items against the real purchased modules
- [ ] 2D space check (top and side views) inside the chassis, with real measurements
- [ ] Firmware (main ESP32-S3 and ESP32-CAM), using the pin map above

## Open items

These need the real, purchased parts. See [`CLAUDE.md`](CLAUDE.md) §20 for details.

1. Photos and measurements of the ESP32-S3 board if it is a clone (row spacing, RGB LED revision).
2. Pin order of the purchased TB6612FNG breakout (the SparkFun order was assumed). The layout depends on it.
3. Photo and measurements of the MP1584 module (its footprint is provisional).
4. GY-530 photos: the XSHUT voltage level (the modules already include the I2C pull-ups).
5. Measured stall current of the TT motor, to validate the driver, switch and wiring.
6. Green LED forward voltage, to set the final value of R5.
7. ESP32-CAM silkscreen, to confirm the pin order, and which end has the header pins.
8. The photo of the chassis shows that the **110 mm side is the front** (between the wheels) and the 140 mm side runs front to back. Still to check: does the 140 mm length fit between the front concave area and the caster, and does the USB cable at the rear edge clear the caster and the chassis?
9. The ESP32-CAM is at the front of the board, 14.5 mm to the left of the centre. The camera flex must reach the camera at the front of the chassis (10 cm or less is best); 15 mm sideways is no problem for that.
10. Height of the capacitors that will be bought (C1, C3 and C11 are the tallest).
