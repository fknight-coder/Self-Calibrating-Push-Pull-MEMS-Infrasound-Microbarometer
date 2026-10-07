# Calibration Methodology

## ZSPARK-InfraSense Microbarometer Calibration Plan

This document defines the calibration methodology for the ZSPARK-InfraSense high-sensitivity infrasound microbarometer.

The calibration process verifies that the system can accurately measure atmospheric pressure fluctuations within the target range:

```text
Frequency range: 0.01 Hz to 20 Hz
```

The calibration process measures:

- Pressure sensitivity
- Frequency response
- Phase response
- Linearity
- Noise floor
- Temperature drift
- Long-term stability
- Wind-noise-reduction performance
- ADC conversion accuracy
- Pneumatic filter cutoff frequency

---

# 1. Calibration Objectives

| Parameter | What must be verified |
|---|---|
| Frequency response | Sensor responds across 0.01–20 Hz |
| Sensitivity | Output changes predictably with known pressure |
| Phase response | Sensor delay is measured across frequency |
| Linearity | Output remains proportional to pressure |
| Noise floor | Smallest measurable pressure variation |
| Pneumatic cutoff | Backing chamber/capillary response near 0.01 Hz |
| Temperature drift | Offset/sensitivity variation with temperature |
| Stability | Offset and noise remain stable over time |
| Wind reduction | Pipe rosette reduces turbulent outdoor noise |

---

# 2. Calibration Setup

```text
Laptop / Signal Generator
           │
           ▼
Speaker Amplifier / Piston Driver
           │
           ▼
Low-Frequency Pressure Actuator
           │
           ▼
Rigid Sealed Calibration Chamber
           │
           ├── Reference Pressure Sensor
           ├── ZSPARK-InfraSense Sensor Under Test
           └── Chamber Temperature Sensor
```

## Required equipment

| Equipment | Purpose |
|---|---|
| Rigid sealed calibration chamber | Produces controlled pressure environment |
| Reference pressure sensor | Measures known pressure waveform |
| ZSPARK sensor under test | Sensor being calibrated |
| Speaker + amplifier | Generates pressure waves mainly above 0.5 Hz |
| Motorized syringe/piston | Generates stable pressure waves below 1 Hz |
| Function generator / laptop | Generates sine, sweep and step signals |
| Oscilloscope | Checks analog output and timing |
| Laptop dashboard | Records waveform and calculates response |
| Temperature sensor | Monitors chamber temperature |
| MicroSD logger | Stores raw sensor data |

---

# 3. Calibration Frequencies

The following frequencies should be tested:

```text
0.01 Hz
0.02 Hz
0.05 Hz
0.1 Hz
0.2 Hz
0.5 Hz
1 Hz
2 Hz
5 Hz
10 Hz
15 Hz
20 Hz
```

## Recommended actuator selection

| Frequency range | Recommended actuator |
|---|---|
| 0.01–0.5 Hz | Motorized syringe / piston |
| 0.5–5 Hz | Piston or large speaker |
| 5–20 Hz | Speaker + amplifier |
| Step response | Syringe/piston volume step |

---

# 4. Calibration Procedure

## 4.1 Pre-calibration checks

Before calibration:

```text
1. Inspect pneumatic tubing for damage.
2. Verify all fittings are leak-tight.
3. Confirm capillary cartridge is installed.
4. Confirm backing chamber is sealed.
5. Check sensor power supply voltage.
6. Check ADC communication.
7. Verify GPS/RTC time.
8. Confirm microSD logging works.
9. Warm up electronics for at least 15–30 minutes.
10. Record sensor and chamber temperature.
```

---

## 4.2 Zero-offset calibration

### Objective

Measure the sensor offset when both pressure ports experience equal pressure.

### Setup

```text
Connect P1 and P2 to the same stable pressure volume.
No real differential pressure should exist.
```

### Procedure

```text
1. Connect both ports to equal pressure.
2. Start data acquisition.
3. Record at least 1 hour of raw waveform data.
4. Record temperature and humidity.
5. Calculate mean output pressure.
6. Store this value as zero offset.
```

### Offset equation

```text
Offset = Mean(Measured Pressure)
```

### Output

```text
- Zero offset in Pa
- Offset standard deviation
- RMS noise
- Temperature during test
```

---

## 4.3 Sensitivity calibration

### Objective

Determine sensor output per unit pressure.

### Procedure

```text
1. Apply a known sinusoidal pressure amplitude.
2. Measure reference pressure amplitude.
3. Measure sensor output amplitude.
4. Repeat at multiple amplitudes.
5. Calculate sensor sensitivity.
```

### Suggested pressure amplitudes

```text
0.01 Pa
0.05 Pa
0.1 Pa
0.5 Pa
1 Pa
5 Pa
10 Pa
```

### Sensitivity equations

```text
Sensitivity (V/Pa) = Sensor Output Voltage / Reference Pressure
```

```text
Sensitivity (counts/Pa) = ADC Output Counts / Reference Pressure
```

```text
Sensitivity (Pa/count) = Reference Pressure / ADC Output Counts
```

### Example

```text
Reference pressure = 1 Pa
Sensor output = 20 mV

Sensitivity = 20 mV/Pa
```

---

## 4.4 Frequency-response calibration

### Objective

Measure amplitude response across the full 0.01–20 Hz operating band.

### Procedure

```text
1. Apply a sine wave at 0.01 Hz.
2. Record reference and test-sensor data.
3. Measure output amplitude.
4. Repeat for all calibration frequencies.
5. Plot frequency versus normalized amplitude.
```

### Gain equation

```text
Gain(dB) = 20 × log10(Measured Amplitude / Reference Amplitude)
```

### Expected output

```text
Frequency-response graph:
Frequency (Hz) on X-axis
Gain (dB) on Y-axis
```

### Target

```text
Useful response from approximately 0.01 Hz to 20 Hz.
```

> The final flatness specification must be based on actual measured data, not assumed from theoretical capillary calculations.

---

## 4.5 Phase-response calibration

### Objective

Measure phase delay between the reference sensor and the ZSPARK sensor.

### Procedure

```text
1. Apply sinusoidal pressure at each frequency.
2. Record reference waveform.
3. Record ZSPARK waveform.
4. Measure time delay between matching waveform peaks.
5. Convert time delay to phase difference.
```

### Phase equation

```text
Phase Difference (degrees) =
360 × Frequency × Time Delay
```

### Output

```text
Phase-response graph:
Frequency (Hz) on X-axis
Phase difference (degrees) on Y-axis
```

---

## 4.6 Linearity test

### Objective

Verify that sensor output is proportional to pressure.

### Procedure

```text
1. Select a fixed frequency, such as 1 Hz.
2. Apply multiple known pressure amplitudes.
3. Record sensor output.
4. Plot measured pressure versus reference pressure.
5. Fit a straight line.
6. Calculate measurement error.
```

### Linearity equation

```text
Linearity Error (%) =
[(Measured Pressure - Reference Pressure) / Reference Pressure] × 100
```

### Output

```text
- Pressure input vs measured pressure graph
- Linearity error percentage
- Maximum error across tested range
```

---

# 5. Pneumatic Cutoff Calibration

## Objective

Determine the actual low-frequency cutoff created by the backing chamber and capillary.

## Procedure

```text
1. Install one capillary cartridge.
2. Apply a low-frequency pressure sweep.
3. Measure response from 0.005 Hz to 1 Hz.
4. Identify the frequency where amplitude drops by approximately 3 dB.
5. Record this as the pneumatic cutoff frequency.
6. Repeat for every capillary cartridge.
```

## Expected output

| Cartridge | Chamber Volume | Capillary ID | Capillary Length | Measured Cutoff |
|---|---:|---:|---:|---:|
| A | TBD | TBD | TBD | TBD |
| B | TBD | TBD | TBD | TBD |
| C | TBD | TBD | TBD | TBD |
| D | TBD | TBD | TBD | TBD |

---

# 6. Noise-Floor Measurement

## Objective

Measure intrinsic sensor, analog front-end and ADC noise.

## Setup

```text
P1 and P2 connected to the same stable pressure volume.

No intentional differential pressure is applied.
```

## Procedure

```text
1. Record raw data for at least 1 hour.
2. Repeat for 24 hours for engineering characterization.
3. Apply calibration coefficients.
4. Calculate RMS pressure noise.
5. Calculate power spectral density.
6. Repeat with different capillary configurations.
```

## RMS equation

```text
P_RMS = sqrt(sum(P[n]^2) / N)
```

## Analyze these frequency bands

| Frequency band | Purpose |
|---|---|
| 0.01–0.1 Hz | Long-period drift/noise |
| 0.1–1 Hz | Atmospheric infrasound region |
| 0.5–2 Hz | Common sensor-comparison region |
| 1–5 Hz | Event and explosion region |
| 5–20 Hz | Upper infrasound region |

---

# 7. Temperature Calibration

## Objective

Measure sensor offset and sensitivity variation with temperature.

## Suggested temperature points

```text
0°C
10°C
20°C
25°C
35°C
45°C
50°C
```

## Procedure

```text
1. Stabilize sensor at each test temperature.
2. Equalize sensor pressure ports.
3. Record zero offset.
4. Apply known calibration pressure.
5. Measure sensitivity.
6. Record sensor-board and backing-chamber temperatures.
7. Fit offset and sensitivity correction curves.
```

## Offset model

```text
Offset(T) = a0 + a1 × T + a2 × T²
```

## Compensation model

```text
P_corrected = [P_measured - Offset(T)] / Sensitivity(T)
```

---

# 8. Stability Test

## Objective

Measure long-term offset, noise and sensitivity stability.

## Procedure

```text
1. Install sensor in stable indoor or sheltered outdoor condition.
2. Record data continuously for 24 hours.
3. Repeat for 7 days.
4. Record temperature, humidity, battery voltage and wind speed.
5. Track offset, RMS noise and calibration drift.
```

## Drift equation

```text
Drift = (Offset_end - Offset_start) / Time Duration
```

## Report drift as

```text
mPa/hour
Pa/day
mPa/degree-Celsius
```

---

# 9. Wind-Noise-Reduction Test

## Objective

Measure effectiveness of the pipe rosette.

## Test sequence

```text
Test A: Bare sensor inlet
Test B: Foam / basic windscreen
Test C: Single pipe inlet
Test D: Four-arm pipe rosette
Test E: Eight-arm pipe rosette
```

## Procedure

```text
1. Deploy sensor outdoors.
2. Measure wind speed using anemometer.
3. Record pressure waveform under similar conditions.
4. Calculate PSD for each configuration.
5. Compare noise level at different frequencies.
```

## Noise-reduction equation

```text
Noise Reduction (dB) =
20 × log10(Bare Inlet Noise / Rosette Noise)
```

---

# 10. Calibration Deliverables

The final calibration report must include:

```text
1. Sensor identification number
2. Hardware configuration
3. Sensor range
4. Capillary cartridge used
5. Backing-chamber volume
6. ADC sample rate
7. Calibration date
8. Temperature during calibration
9. Sensitivity value
10. Frequency-response graph
11. Phase-response graph
12. Noise PSD graph
13. RMS noise table
14. Linearity plot
15. Temperature compensation curve
16. Stability plot
17. Wind-noise-reduction comparison
18. Calibration coefficient JSON file
```

---

# 11. Calibration Status Template

```text
Sensor ID:
Date:
Operator:
Pressure Sensor Model:
ADC Model:
STM32 Firmware Version:
Capillary Cartridge:
Backing Chamber Volume:
Sample Rate:
Reference Sensor:
Temperature:
Humidity:

Frequency Response Test: Pending / Complete
Sensitivity Test: Pending / Complete
Phase Test: Pending / Complete
Noise Test: Pending / Complete
Temperature Test: Pending / Complete
Stability Test: Pending / Complete
Wind Rosette Test: Pending / Complete

Final Calibration Status:
```

---

# 12. Important Notes

```text
1. Do not claim 0.01 Hz response without measured calibration data.
2. Do not claim millipascal noise performance without PSD/RMS measurement.
3. Store raw data before applying filters.
4. Use the same calibration configuration during field deployment.
5. Recalibrate after capillary, chamber, sensor, gain or ADC configuration changes.
6. Recalibrate after major mechanical shock, water ingress or sensor replacement.
7. Keep calibration data version-controlled in config/calibration_config.json.
```
