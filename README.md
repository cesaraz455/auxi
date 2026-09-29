# auxi — Soccer Robot Controller Board

**auxi** is a small soccer robot: a first-semester university project. This repository holds the KiCad design for its single custom PCB.

> **Status: Stage 1 (schematic).** The schematic is complete and passes ERC with no errors or warnings. It is **preliminary** until the purchased modules are checked against their reference pinouts (see [Open items](#open-items)). PCB layout has **not** started.

---

## The robot

- **Drive:** 2 yellow TT gear motors plus 1 rear caster wheel.
- **Chassis:** 3D-printed, about 150 × 150 mm. The front has a built-in **concave area that guides the ball**.
- **Two-level layout:**
  - **Bottom level (chassis):** batteries, motors, caster and wiring.
  - **Top level:** one **90 × 100 mm PCB** on four M3 standoffs, sitting above the batteries and motors.
- **Main MCU:** ESP32-S3-DevKitC-1 N16R8. It reads the sensors, drives the motors and LEDs, and receives commands from an **Xbox controller over Bluetooth Low Energy**. The ESP32-S3 has no Classic Bluetooth.
- **Camera:** AI Thinker ESP32-CAM. It streams live video over Wi-Fi to a phone and shows the distances it receives from the main ESP32 over UART. The OV2640 camera is mounted at the front of the chassis on a long flex cable.
- **Sensors:** 3 × VL53L0X time-of-flight sensors (GY-530 modules) in the concave front area: FRONT (0°), LEFT (+45°) and RIGHT (−45°).
- **Proximity LEDs:** 3 LEDs whose colors are set by the university rules. Each LED turns on when its sensor reads **≤ 100 mm**.

## What is on the PCB

| On the PCB (plug-in modules and parts) | Off the PCB (connects through a connector) |
|---|---|
| ESP32-S3-DevKitC-1 (2 × 1×22 female headers) | 2S 18650 battery (XT30) |
| ESP32-CAM (2 × 1×8 female headers) | Power switch (screw terminal) |
| TB6612FNG breakout (female headers) | Blue power LED (2-pin header) |
| MP1584EN mini buck module (5.0 V) | 2 TT motors (screw terminals) |
| Capacitors, resistors, Schottky diode | 3 VL53L0X sensors (JST-XH 5P) |
| All connectors | 3 proximity LEDs (2-pin headers) |
| | OV2640 camera (flex cable into the ESP32-CAM) |

## Power architecture

```
BAT 2S (6.0–8.4 V) ─ XT30 ─ external switch ─ +VBAT_SW
+VBAT_SW ─┬─ R1 ─ blue power LED
          ├─ bulk caps ─ TB6612 VM (motors)
          └─ MP1584 ─ +5V ─┬─ D1 Schottky ─ +5V_ESP ─ ESP32-S3 5V pin
                           └─ ESP32-CAM 5V pin
ESP32-S3 3V3 pin ─ +3V3 ─ VL53L0X sensors, TB6612 logic, I2C pull-ups (DNP)
```

- Nothing connects to the battery before the switch. When the switch is off, the whole robot is off.
- D1 keeps the USB port and the robot from powering each other while you program the board.
- The firmware **must** limit motor PWM to **70 %**, which gives about 5.9 V average at 8.4 V.

## Proximity LEDs (university rules)

| Sensor | LED color | Connector |
|---|---|---|
| FRONT | Red | `J_LED_FRONT1` |
| LEFT | Yellow | `J_LED_LEFT1` |
| RIGHT | Green | `J_LED_RIGHT1` |

A LED is **on** when its sensor reads ≤ 100 mm. It is **off** when the reading is above 100 mm, invalid or out of range. The firmware may average readings, but it must not use hysteresis that keeps a LED on above 100 mm.

## Repository layout

| Path | Contents |
|---|---|
| `auxi.kicad_pro` | KiCad project |
| `auxi.kicad_sch` | Schematic (Stage 1 deliverable) |
| `auxi.kicad_pcb` | PCB (empty until Stage 2) |
| `auxi.kicad_sym` | Project symbol library (module header rows, etc.) |
| `auxi.pretty/` | Project footprint library (`MP1584_Mini_PROVISIONAL` has estimated pads) |
| `sym-lib-table`, `fp-lib-table` | Project library tables |
| `docs/auxi_esquematico.pdf` | Schematic PDF export |
| `docs/auxi_bom_pcb.csv` | Preliminary PCB BOM |

The full design specification (confirmed decisions, power, pinout, connectors, capacitors, PCB rules and open items) is in [`CLAUDE.md`](CLAUDE.md). It is the source of truth for every design decision.

## Opening the project

1. Install **KiCad 8 or newer**.
2. Open `auxi.kicad_pro`. The project libraries (`auxi`) load through the local `sym-lib-table` and `fp-lib-table` using `${KIPRJMOD}`.

## Assembly warnings

1. **Set the MP1584 to 5.0 V with a multimeter BEFORE plugging in any module.**
2. **Check battery polarity** (`BAT+` / `BAT−`) before connecting it.
3. **The switch path carries all of the robot's current.** Use 20 AWG wire and the screw terminal. Never use Dupont jumpers there.
4. **Remove the ESP32-CAM to program it**, using the ESP32-CAM-MB programmer base.
5. **Motor PWM must never exceed 70 %.**
6. **Use protected 18650 cells**, or charge them outside the robot with a balancing 2S charger. The PCB has no BMS or charging circuit.

## Roadmap

- [x] **Stage 1:** schematic, footprints and ERC
- [ ] Close the open items against the real purchased modules
- [ ] **Stage 2:** 2D space check (top and side views) inside the chassis
- [ ] **Stage 3:** 90 × 100 mm 2-layer PCB layout, DRC and mechanical verification
- [ ] Firmware (main ESP32-S3 and ESP32-CAM)

## Open items

These need the real, purchased parts. See [`CLAUDE.md`](CLAUDE.md) §20 for details.

1. Photos and measurements of the ESP32-S3 board if it is a clone (row spacing, RGB LED revision).
2. Pin order of the purchased TB6612FNG breakout (the SparkFun order was assumed).
3. Photo and measurements of the MP1584 module (its footprint is provisional).
4. GY-530 photos: whether R3/R4 pull-ups are needed, and the XSHUT voltage level.
5. Measured stall current of the TT motor, to validate the driver, switch and wiring.
6. Green LED forward voltage, to set the final value of R7.
7. ESP32-CAM silkscreen, to confirm the pin order.
8. Which XT30UPB-M pin is `+` in the footprint.
