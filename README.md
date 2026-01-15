# FREMO INTELIGENT MODULE OCCUPANCY DETECTOR
## DCC Track Occupancy Detector
## 2-Channel Block Occupancy Detector for Model Railways

---

## Overview

This board is a simple and reliable solution for detecting whether a track section is occupied by rolling stock. The board is designed for DCC systems (Digital Command Control) and provides a digital signal (high/low) to a microcontroller such as ESP32.

---

## Specifications

| Parameter | Value |
|-----------|-------|
| Number of channels | 2 |
| Supply voltage | 3.3V DC |
| DCC voltage | 12-16V (N, TT, H0) |
| Output | Open-collector, active low |
| Microcontroller | ESP32 or similar (3.3V logic) |
| Current transformer | See compatible types below |

---

## Compatible Current Transformers

The board can be used with several different current transformer types from the ZHT series. Choose based on availability and price - all work equally well after adjusting the burden resistor.

| Current Transformer | Ratio | Burden Resistor (R100/R200) | Notes |
|---------------------|-------|----------------------------|-------|
| **ZHT102** | 1:2000 | **220Ω** | Good stock at JLCPCB |
| **ZHT103** | 1:1000 | **100Ω** | Slightly better signal/noise |
| **ZHT108T** | 1:1000 | **100Ω** | Alternative to ZHT103 |

**Important:** All other components remain unchanged - only the burden resistor (R100/R200) needs to be adjusted for the selected transformer.

### Calculating Burden Resistor for Other Transformer Types

If you want to use a different current transformer, the burden resistor can be calculated:

```
R_burden = (Desired_voltage × Turns_ratio) / Primary_current

Example for 10mV at 100mA primary current:
- ZHT102 (1:2000): R = (10mV × 2000) / 100mA = 200Ω → use 220Ω
- ZHT103 (1:1000): R = (10mV × 1000) / 100mA = 100Ω → use 100Ω
```

---

## 5V Compatibility

The board is designed for 3.3V but **works with 5V without any modifications**.

### Why It Works at Both Voltages

| Component | At 3.3V | At 5V | Notes |
|-----------|---------|-------|-------|
| LM358 (opamp) | Limited headroom | Better headroom | Actually performs better at 5V |
| LM393 (comparator) | OK | OK | Works well at both voltages |
| LED brightness | Dimmer (~1.4mA) | Brighter (~3mA) | Both acceptable |
| Threshold range | 0 to ~1.1V | 0 to ~1.7V | Scales automatically |

### Optional Modifications for 5V

If you want to optimize for 5V operation, you can make these changes:

| Component | 3.3V Value | 5V Optimized | Reason |
|-----------|------------|--------------|--------|
| R112/R212 (LED) | 1kΩ | 680Ω | Brighter LED |
| R106/R206 (gain) | 100kΩ | 68kΩ | More headroom (optional) |

**These changes are optional** - the board works fine at 5V with the standard component values.

### Microcontroller Compatibility

| Microcontroller | VCC | Compatible? |
|-----------------|-----|-------------|
| ESP32 | 3.3V | ✅ Yes (native) |
| ESP8266 | 3.3V | ✅ Yes (native) |
| Arduino Uno/Nano | 5V | ✅ Yes |
| Arduino Mega | 5V | ✅ Yes |
| Raspberry Pi Pico | 3.3V | ✅ Yes (native) |
| STM32 (most) | 3.3V | ✅ Yes (native) |

**Important:** Always match VCC to your microcontroller's logic level. Do not mix 5V VCC with 3.3V-only GPIO pins without level shifting.

---

## Operating Principle

1. **Current transformer** (ZHT102/ZHT103) measures current through the track without breaking the DCC signal
2. **Rectifier** (1N5819WS diodes) converts the AC signal to DC
3. **Amplifier** (LM358) amplifies the weak signal
4. **Comparator** (LM393) compares against an adjustable threshold and provides a clean digital output
5. **LED indicator** visually shows whether the track is occupied

---

## Detection Capability

The board can detect:

- ✅ Locomotives with motor (100-500mA)
- ✅ Coaches with LED lighting (10-50mA)
- ✅ Coaches with wheel resistors (down to ~5-10mA)

---

## Wiring

### Current Transformer (ZHT102/ZHT103/ZHT108T)

The current transformer works by sensing the magnetic field created by current flowing through its center hole. **No electrical connection is made to the DCC wire** - the wire simply passes through the hole.

#### Installation Steps

1. **Identify one rail feeder wire** going to the track section you want to monitor
2. **Pass the DCC wire through the center hole** of the current transformer
3. **Connect the transformer's secondary wires** (2 pins) to the board's input (S10/S11 or S20/S21)

```
                    Current Transformer
                    ┌─────────────────┐
                    │    ┌───────┐    │
DCC Rail Feeder ════════╡ HOLE  ╞════════► To Track Section
  (from booster)    │    └───────┘    │     (Rail 1)
                    │                 │
                    │  ○         ○    │← Secondary pins
                    └──┼─────────┼────┘   (to S10/S11)
                       │         │
                       ▼         ▼
                    To detector board
```

**Important notes:**
- The DCC wire passes **through** the hole - it does not connect to the transformer electrically
- Only **one rail** needs to pass through the transformer (not both)
- The other rail continues directly to the track without passing through any transformer

#### Increasing Sensitivity

For very weak loads (high-resistance wheel sensors), you can increase sensitivity by passing the wire through the hole **multiple times**:

```
                    ┌─────────────────┐
                    │    ┌───────┐    │
              ══════════╡       ╞═══╗
                    │    │       │   ║
              ╔═════════╡       ╞═══╝
              ║     │    └───────┘    │
 From DCC ════╝     │                 │      To Track
                    └─────────────────┘

 2 turns = 2× signal strength
```

| Turns through hole | Signal multiplication |
|-------------------|----------------------|
| 1 (standard) | 1× |
| 2 | 2× |
| 3 | 3× |

**Note:** Multiple turns are rarely needed for normal operation with locomotives and lit coaches.

#### Complete Wiring Diagram

```
DCC Booster                                          Track Section
     │                                                    │
Rail A ───────────────────────────────────────────────── Rail A
     │                                                    │
     │         ┌─────────────────┐                        │
     │         │    ┌───────┐    │                        │
Rail B ════════════╡  HOLE  ╞════════════════════════════ Rail B
               │    └───────┘    │
               │                 │
               │  ○         ○    │
               └──┼─────────┼────┘
                  │         │
                  ▼         ▼
               S10/S11 on detector board
               (or S20/S21 for channel 2)
```

### Power Supply

| Pin | Connection |
|-----|------------|
| VCC | 3.3V from ESP32 |
| GND | GND from ESP32 |

### Output to ESP32

| Pin | Function | ESP32 GPIO |
|-----|----------|------------|
| CMP1 | Channel 1 output | Any GPIO |
| CMP2 | Channel 2 output | Any GPIO |

**Output logic:**
- **LOW (0V)** = Track occupied (LED on)
- **HIGH (3.3V)** = Track free (LED off)

---

## Calibration

Each channel has a trimmer potentiometer (VR1/VR2) for sensitivity adjustment:

1. Ensure the track is **free** (no rolling stock)
2. Turn the potentiometer until the LED **turns off**
3. Place a coach with lighting or wheel resistor on the track
4. Adjust the potentiometer until the LED **turns on reliably**
5. Test with different rolling stock

**Tip:** Start with the potentiometer turned towards GND (low threshold) and slowly turn up until false detections disappear.

---

## Bill of Materials

| Component | Value | Quantity | Notes |
|-----------|-------|----------|-------|
| Current transformer | ZHT102, ZHT103 or ZHT108T | 2 | See table above |
| R100, R200 | 220Ω (ZHT102) / 100Ω (ZHT103/ZHT108T) | 2 | Burden resistor - adjust for chosen transformer |
| R101, R102, R201, R202 | 75Ω | 4 | Current limiting |
| R103, R203 | 100kΩ | 2 | Discharge |
| R104, R204 | 22kΩ | 2 | Amplifier input |
| R106, R206 | 100kΩ | 2 | Amplifier feedback |
| R107, R207 | 1kΩ | 2 | Bias |
| R108, R208 | 10kΩ | 2 | Threshold divider |
| R109, R209 | 1kΩ | 2 | Threshold divider |
| R110, R210 | 10kΩ | 2 | Pull-up |
| R111, R211 | 1MΩ | 2 | Hysteresis |
| R112, R212 | 1kΩ | 2 | LED series resistor |
| VR1, VR2 | 5kΩ | 2 | Trimmer potentiometer |
| C11, C21 | 10µF | 2 | Smoothing |
| C1, C2 | 100nF | 2 | Decoupling |
| D11, D12, D21, D22 | 1N5819WS | 4 | Schottky diodes |
| LED3, LED2 | KT-0603R | 2 | Red LED 0603 |
| U1 | LM358DR2G | 1 | Dual opamp |
| U2 | LM393DR2G | 1 | Dual comparator |

---

## Troubleshooting

| Problem | Possible Cause | Solution |
|---------|----------------|----------|
| LED always on | Threshold too low | Turn VR up |
| LED never on | Threshold too high / no connection | Turn VR down / check wiring |
| Unstable detection | Load too weak / noise | Add wheel resistors to coaches |
| No response | Transformer wired incorrectly | Check that DCC wire passes through hole |

---

## Expansion

The board can easily be combined with multiple units to monitor an entire layout. Use an ESP32 with sufficient GPIO pins and connect multiple detectors in parallel.

---

## License

This design is freely available for private use in the model railway community.

Happy modelling! 🚂
