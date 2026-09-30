# Innovation and Unique Selling Proposition

## ApeX-InfraSense: What Makes Our Solution Different

ApeX-InfraSense is not designed as only a basic pressure sensor or a normal digital barometer.

It is designed as a complete intelligent infrasound-monitoring platform that combines:

```text
Differential Pressure Sensing
+ Tunable Pneumatic Filtering
+ Wind-Noise Reduction
+ 24-bit Synchronous Digitization
+ Embedded STM32 Signal Processing
+ Environmental Sensor Fusion
+ Calibration and Stability Validation
+ Future Force-Feedback Self-Test Path
```

The main objective is to measure atmospheric pressure fluctuations in the following infrasound range:

```text
0.01 Hz to 20 Hz
```

while maintaining low noise, low drift, long-term stability, and high signal fidelity.

---

# 1. Innovation Summary

| Innovation | Description | Technical Benefit |
|---|---|---|
| Differential pneumatic measurement | Measures pressure difference between atmospheric input P1 and filtered reference P2 | Rejects slow common atmospheric-pressure variation |
| Tunable capillary filter | Uses interchangeable capillary cartridges and backing chambers | Allows experimental tuning of lower cutoff near 0.01 Hz |
| Pipe-rosette wind filter | Uses multiple distributed pneumatic inlets | Reduces local turbulent wind-pressure noise |
| 24-bit synchronous acquisition | Uses multi-channel delta-sigma ADC | Captures weak pressure, vibration and diagnostic signals together |
| STM32 edge intelligence | Performs acquisition, filtering, FFT, RMS and event detection locally | Reduces dependence on external computing hardware |
| Environmental sensor fusion | Combines pressure, wind, vibration, temperature and humidity data | Reduces false alarms and improves event confidence |
| GPS PPS synchronization | Uses precise time reference | Enables future multi-sensor array operation |
| Embedded health monitoring | Monitors sensor, ADC, SD card, power and environmental state | Improves field reliability |
| Electrical self-calibration path | Future actuator/current-injection calibration design | Enables drift and sensitivity verification in the field |
| Array-ready architecture | Supports multiple synchronized nodes | Enables direction finding and event localization |

---

# 2. Core USP

## 2.1 Differential Measurement Instead of Ordinary Barometric Measurement

Ordinary pressure sensors measure absolute atmospheric pressure.

Kestrel-InfraSense measures the difference between:

```text
P1 = Wind-filtered atmospheric pressure
P2 = Controlled backing-chamber reference pressure
```

```text
Delta-P = P1 - P2
```

This is important because the desired infrasound waveform is very small compared with normal atmospheric pressure.

```text
Normal atmospheric pressure: approximately 101,325 Pa

Desired infrasound fluctuation:
millipascal level to several pascals
```

### Value

```text
Differential measurement allows the system to focus on
small pressure changes rather than the full atmospheric pressure.
```

---

## 2.2 Tunable Long-Period Pneumatic Equalization

The system uses:

```text
Rigid Backing Chamber
        +
Fine Capillary Restriction
        =
Pneumatic High-Pass Filter
```

The capillary and chamber allow very slow weather-pressure changes to equalize while preserving faster infrasonic pressure fluctuations.

```text
Fast infrasound wave:
P1 changes quickly
P2 changes slowly
Delta-P is measured

Slow weather-pressure drift:
P1 changes slowly
P2 gradually equalizes
Delta-P becomes small
```

The approximate pneumatic cutoff model is:

```text
fc = 1 / (2 × pi × R × C)
```

Where:

| Variable | Meaning |
|---|---|
| `fc` | Pneumatic cutoff frequency |
| `R` | Pneumatic resistance of capillary |
| `C` | Pneumatic compliance of backing chamber |

### Innovation

Instead of a permanently fixed capillary, ApeX-InfraSense uses interchangeable capillary cartridges.

```text
Capillary A → Higher cutoff response
Capillary B → General measurement response
Capillary C → Lower-frequency response
Capillary D → Ultra-low-frequency response
```

### Value

```text
The sensor can be experimentally tuned for the best response
near the 0.01 Hz target instead of relying only on theoretical calculations.
```

---

# 3. Wind-Noise Intelligence

## 3.1 Pipe-Rosette Spatial Filtering

Wind turbulence is one of the major difficulties in outdoor infrasound measurement.

ApeX-InfraSense uses a multi-arm pipe rosette.

```text
          Protected Inlet
                │
                ▼
          ── Pipe Arm ──

Protected ──────┬────── Pipe Arm ──────┐
Inlet           │                       │
                ├────── Pipe Arm ──────┼── Central Manifold
                │                       │
Protected ──────┴────── Pipe Arm ──────┘
Inlet
```

### Working principle

```text
Wind turbulence:
- Local
- Random
- Small spatial scale
- Different at each inlet

Infrasound:
- Long wavelength
- Coherent across all inlets
- Preserved after pressure averaging
```

### Value

```text
The pipe rosette reduces wind-generated pressure noise before
the signal reaches the pressure sensor and ADC.
```

---

## 3.2 Adaptive Wind and Vibration Validation

The system does not treat every pressure waveform as a real event.

It uses environmental references:

```text
Pressure waveform
+ Wind-speed measurement
+ Vibration measurement
+ Temperature trend
+ Secondary pressure reference
= Event confidence score
```

### Decision logic

| Observed condition | System interpretation |
|---|---|
| High pressure energy + low wind + low vibration | Possible real infrasonic event |
| High pressure energy + strong wind correlation | Probable wind turbulence |
| High pressure energy + strong vibration correlation | Probable mechanical artifact |
| Similar signal on primary and secondary pressure channels | Higher event confidence |
| Large temperature change + slow baseline shift | Possible thermal drift |
| Sudden offset + humidity increase | Possible moisture or pneumatic issue |

### Value

```text
The system is not only a data logger.
It actively validates whether the detected pressure signal is likely real.
```

---

# 4. High-Resolution Synchronous Measurement

## 4.1 Multi-Channel 24-bit Acquisition

The proposed acquisition architecture uses a 24-bit simultaneous-sampling ADC.

```text
CH0 → Main differential pressure waveform
CH1 → Reference / secondary pressure waveform
CH2 → Vibration reference
CH3 → Calibration or diagnostic signal
```

### Why synchronous channels are valuable

```text
Pressure, vibration and calibration channels are recorded at the same time.
```

This supports:

- Pressure-to-vibration correlation
- Secondary-sensor comparison
- Self-test verification
- Fault diagnosis
- Multi-channel event validation
- Future array synchronization

### Value

```text
Synchronous data acquisition allows the system to identify
whether a waveform comes from atmosphere, wind, vibration,
electronics or a calibration source.
```

---

# 5. Standalone Embedded Intelligence

## 5.1 STM32H743-Based Architecture

The STM32H743 is used as the main controller.

```text
ADS131M04 ADC
      │
      ▼
STM32H743
      │
      ├── SPI + DMA acquisition
      ├── GPS PPS timestamps
      ├── microSD waveform logging
      ├── Temperature compensation
      ├── RMS and peak measurement
      ├── FFT and PSD calculation
      ├── STA/LTA event triggering
      ├── USB live waveform streaming
      ├── Environmental sensor reading
      └── System-health monitoring
```

### Value

```text
The primary sensor can operate without a Raspberry Pi.
This reduces power consumption, improves deterministic timing,
and lowers digital-noise risk near sensitive analog electronics.
```

---

## 5.2 Edge-Based Event Detection

The embedded system uses continuous pressure monitoring.

```text
Raw pressure waveform
        │
        ▼
Band-pass filtering
        │
        ▼
RMS / energy calculation
        │
        ▼
STA/LTA event trigger
        │
        ▼
Environmental correlation
        │
        ▼
Event storage + alert + confidence score
```

### Trigger concept

```text
STA/LTA = Short-Term Average / Long-Term Average
```

When the short-term energy becomes much higher than normal background energy, the system stores:

```text
- Pre-event waveform
- Event waveform
- GPS time
- RMS pressure
- Peak pressure
- Dominant frequency
- Wind speed
- Vibration level
- Temperature
- Event confidence
```

### Value

```text
Instead of continuously sending all data, the system can identify
important events and preserve high-value event data automatically.
```

---

# 6. Temperature Compensation and Stability Monitoring

## 6.1 Multi-Point Temperature Sensing

The system uses multiple temperature sensors.

```text
Temperature Sensor 1 → Pressure sensor / analog PCB
Temperature Sensor 2 → Backing chamber
Temperature Sensor 3 → Enclosure electronics
```

Temperature can affect:

- Sensor offset
- Sensor sensitivity
- Amplifier offset
- ADC reference stability
- Capillary flow resistance
- Backing chamber pressure
- Mechanical dimensions

## 6.2 Compensation Model

```text
P_corrected = [P_measured - Offset(T)] / Sensitivity(T)
```

A simple offset model can be:

```text
Offset(T) = a0 + a1 × T + a2 × T²
```

### Value

```text
The system measures and corrects thermal drift instead of
assuming the sensor output remains constant across temperature changes.
```

---

# 7. Embedded Sensor Health Monitoring

The system continuously checks its own operating condition.

| Health parameter | What it indicates |
|---|---|
| ADC communication status | ADC connection and acquisition health |
| ADC saturation flag | Signal range or gain problem |
| GPS PPS lock | Timestamp accuracy |
| SD-card write status | Continuous recording reliability |
| Battery voltage/current | Power health |
| Temperature trend | Thermal drift risk |
| Humidity trend | Moisture/condensation risk |
| Wind-speed signal | Wind-noise risk |
| Vibration amplitude | Mechanical interference |
| Secondary sensor agreement | Sensor reliability |
| Pressure baseline shift | Leakage, blockage or drift risk |

### Value

```text
The system can report sensor health before a measurement failure
becomes a major data-quality problem.
```

---

# 8. Future Force-Feedback Self-Calibration

> This section describes the future innovation path. It should be treated as an advanced module until physically implemented and calibrated.

## 8.1 Force-Feedback Principle

```text
Pressure Difference
        │
        ▼
Bellows / Diaphragm Deflection
        │
        ▼
Capacitive / Optical Position Sensor
        │
        ▼
Position Error
        │
        ▼
STM32 PID Controller
        │
        ▼
DAC + Current Driver
        │
        ▼
Voice-Coil / Magnetic Actuator
        │
        ▼
Restoring Force Returns Sensing Element Near Zero Position
```

## 8.2 Force-feedback equations

```text
F_feedback = Kf × I_feedback
```

```text
Delta-P = F_feedback / A
```

```text
Delta-P = (Kf × I_feedback) / A
```

Where:

| Term | Meaning |
|---|---|
| `F_feedback` | Actuator restoring force |
| `Kf` | Force constant |
| `I_feedback` | Feedback current |
| `A` | Effective diaphragm/bellows area |
| `Delta-P` | Equivalent pressure difference |

## 8.3 Expected advantages

- Keeps diaphragm/bellows close to its null position
- Improves mechanical linearity
- Reduces displacement-related nonlinearity
- Reduces hysteresis
- Extends dynamic range
- Enables electrical self-test
- Provides actuator-current health information
- Supports in-field sensitivity verification

### Value

```text
Future force feedback can transform the system from a passive
pressure sensor into an active, self-validating microbarometer.
```

---

# 9. Calibration-Ready Design

Kestrel-InfraSense is designed for measurable and repeatable evaluation.

```text
Signal Generator
      │
      ▼
Speaker / Piston / Motorized Syringe
      │
      ▼
Sealed Calibration Chamber
      │
      ├── Reference Sensor
      └── Sensor Under Test
```

## Validation parameters

| Evaluation metric | Proposed method |
|---|---|
| Frequency response | Sine sweep from 0.01 Hz to 20 Hz |
| Sensitivity | Known pressure versus output |
| Phase response | Compare reference and sensor waveform phase |
| Linearity | Test multiple pressure amplitudes |
| Noise floor | Equalize pressure ports and calculate RMS/PSD |
| Temperature drift | Repeat tests at multiple temperatures |
| Stability | Record offset/noise for 24 hours to 30 days |
| Wind reduction | Compare bare inlet and rosette noise PSD |

### Value

```text
The design is not limited to a prototype demonstration.
It includes a complete path to quantify performance scientifically.
```

---

# 10. Array-Ready Future Scope

The design supports future deployment of multiple GPS-synchronized nodes.

```text
              Node 1
                ▲
               / \
              /   \
             /     \
        Node 2 ----- Node 3
```

## Future array capabilities

- Event verification across multiple nodes
- Time-difference-of-arrival measurement
- Direction-of-arrival estimation
- Back-azimuth estimation
- Source-location estimation
- Local wind-noise discrimination
- Multi-station event confidence scoring
- Distributed disaster-monitoring network

### Value

```text
The architecture can grow from one SIH prototype into
a scalable infrasound sensor-array platform.
```

---

# 11. USP Comparison

| Feature | Basic Barometer | Typical Low-Cost Infrasound Sensor | Kestrel-InfraSense |
|---|---|---|---|
| Measures absolute pressure | Yes | Sometimes | Diagnostic only |
| Differential pressure sensing | No | Yes | Yes |
| Tunable pneumatic filter | No | Usually fixed | Yes |
| Pipe-rosette wind reduction | No | Optional | Core design feature |
| 24-bit synchronous channels | No | Limited | Yes |
| Vibration reference | No | Rare | Yes |
| Wind correlation | No | Rare | Yes |
| Temperature compensation | Limited | Basic | Multi-point compensation |
| GPS PPS timing | No | Optional | Included |
| Embedded DSP | No | Limited | STM32-based DSP |
| SD-card waveform logging | No | Sometimes | Included |
| Real-time dashboard | No | Sometimes | Included |
| Self-health monitoring | No | Limited | Included |
| Force-feedback path | No | Rare | Future innovation module |
| Sensor-array readiness | No | Limited | Included |

---

# 12. Final USP Statement

> **Kestrel-InfraSense is a differential, wind-noise-reduced, calibration-ready infrasound microbarometer platform that combines tunable pneumatic filtering, 24-bit synchronous digitization, STM32 edge processing, GPS timing, environmental sensor fusion, and future force-feedback self-validation.**

```text
It is not only a pressure sensor.

It is an intelligent infrasound measurement system designed to:
- Detect weak low-frequency atmospheric pressure signals
- Reject wind, vibration and thermal noise
- Store scientifically useful waveform data
- Validate system health
- Provide real-time event awareness
- Scale into a distributed sensor array
```

---

# 13. Innovation Summary for SIH Presentation

```text
1. Tunable Capillary + Backing Chamber:
   Adjustable low-frequency response near 0.01 Hz.

2. Adaptive Wind/Vibration Rejection:
   Uses environmental sensor fusion to reduce false alerts.

3. 24-bit Synchronous Multi-Channel Acquisition:
   Captures pressure, reference, vibration and calibration data together.

4. STM32 Edge Intelligence:
