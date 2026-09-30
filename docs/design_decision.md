# Design Decisions

## SparkX-InfraSense Engineering Design Decisions

This document explains why specific design choices were selected for the SparkX-InfraSense project.

The purpose is to provide technical justification for the architecture, components and implementation strategy.

---

# 1. Design Goal

The project requires a high-sensitivity microbarometer capable of detecting atmospheric infrasound in the range:

```text
0.01 Hz to 20 Hz
```

The system must also support:

- Low noise
- Long-term stability
- Differential pressure sensing
- Temperature compensation
- Wind-noise reduction
- Calibration
- Real-time data acquisition
- Waveform display and analysis

---

# 2. Differential Pressure Sensor Instead of Absolute Barometer

## Decision

Use a low-range differential pressure sensor as the primary sensing element.

## Why

A normal barometer measures total atmospheric pressure.

```text
Atmospheric pressure is approximately 101,325 Pa.
```

The desired infrasonic pressure signal may be very small.

```text
Desired signal can be millipascals to several pascals.
```

A differential sensor measures:

```text
Delta-P = P1 - P2
```

Where:

```text
P1 = atmospheric pressure from wind-filtered inlet
P2 = filtered backing-chamber reference pressure
```

## Benefit

```text
Differential sensing rejects slow common pressure variations and
focuses measurement on small atmospheric pressure fluctuations.
```

---

# 3. Low-Range Sensor Instead of High-Range Sensor

## Decision

Use a pressure sensor in the approximate range:

```text
±125 Pa to ±250 Pa
```

## Why

A high-range sensor such as ±2 kPa or ±10 kPa has less sensitivity for tiny pressure variations.

## Benefit

| Low-range differential sensor | High-range sensor |
|---|---|
| Better small-signal sensitivity | Better large-pressure tolerance |
| Better ADC range utilization | Weak infrasound uses fewer ADC counts |
| More suitable for microbarometer | More suitable for airflow/general pressure |
| Higher risk of strong-event saturation | Lower risk of saturation |

## Final rationale

```text
±125 Pa to ±250 Pa provides a practical compromise between
weak-signal sensitivity and strong-event headroom.
```

---

# 4. Pneumatic Backing Chamber and Capillary Filter

## Decision

Use a rigid backing chamber and capillary restriction for slow pressure equalization.

## Why

Slow weather pressure changes can saturate or drift a differential sensor.

The capillary and chamber create a pneumatic high-pass behavior.

```text
fc = 1 / (2 × pi × R × C)
```

## Benefit

```text
Slow weather pressure changes equalize through the capillary.
Fast infrasound pressure changes remain different between P1 and P2.
```

## Innovation

Use interchangeable capillary cartridges instead of permanently fixed tubing.

This enables:

- Experimental tuning of lower cutoff frequency
- Comparison of multiple response curves
- Adaptation for laboratory and outdoor testing
- Easier optimization near 0.01 Hz

---

# 5. Pipe Rosette Instead of Single Inlet

## Decision

Use a multi-arm pipe rosette for wind-noise reduction.

## Why

Wind turbulence can create stronger pressure fluctuations than distant infrasound.

A single inlet is highly sensitive to local air turbulence.

## Benefit

```text
Multiple distributed inlets average local wind turbulence,
while long-wavelength infrasound remains coherent.
```

## Implementation strategy

```text
Version 1:
Four equal-length rosette arms

Version 2:
Eight equal-length rosette arms
```

---

# 6. 24-bit ADC Instead of 12-bit ADC

## Decision

Use ADS131M04 24-bit delta-sigma ADC for the main pressure channel.

## Why

The system must detect very small pressure variations while preserving dynamic range for larger events.

A 12-bit ADC has only:

```text
4096 digital codes
```

A 24-bit ADC has:

```text
16,777,216 digital codes
```

## Benefit

```text
24-bit conversion provides much greater quantization headroom,
allowing analog sensor noise instead of ADC quantization to become
the main limitation.
```

## Important note

```text
24-bit nominal resolution does not mean 24 effective bits.
Actual performance depends on sensor noise, analog noise,
reference stability, PCB layout, power supply and environment.
```

---

# 7. ADS131M04 Instead of Basic 16-bit ADC

## Decision

Use ADS131M04 for final multi-channel acquisition.

## Why

The ADS131M04 provides:

```text
- 24-bit delta-sigma conversion
- Four simultaneous channels
- SPI interface
- DRDY output
- Suitable sample rates
- Multi-channel synchronous acquisition
```

## Benefit

```text
Main pressure, secondary pressure, vibration and diagnostic channels
can be recorded at the same time.
```

This improves:

- Noise rejection
- Vibration correlation
- Calibration analysis
- Health diagnostics
- Future array development

---

# 8. STM32H743 Instead of Raspberry Pi as Main Acquisition Controller

## Decision

Use STM32H743 as the main real-time controller.

## Why

The STM32 provides:

- Deterministic interrupt timing
- SPI DMA support
- GPS PPS handling
- microSD logging
- Real-time DSP
- Lower power consumption
- Less digital switching noise near analog front end
- Fast boot time
- Embedded field deployment capability

## Why not Raspberry Pi as main controller?

| Raspberry Pi | STM32H743 |
|---|---|
| Linux operating system | Real-time embedded firmware |
| More software flexibility | Better deterministic timing |
| Higher power consumption | Lower power consumption |
| More digital electrical noise | Better analog isolation |
| Better dashboard capability | Better sensor acquisition reliability |

## Final decision

```text
STM32H743 is the primary acquisition and logging controller.

Laptop/Python dashboard is used for real-time visualization.
```

---

# 9. GPS PPS Instead of Only Network Time

## Decision

Use GPS module with PPS output and DS3231 backup RTC.

## Why

Network time may not be available during outdoor deployment.

GPS PPS provides a precise timing pulse.

## Benefit

```text
GPS PPS enables accurate event timestamps and future
multi-node sensor-array synchronization.
```

---

# 10. microSD Logging Instead of Only Live Streaming

## Decision

Store raw waveform data locally on high-endurance microSD.

## Why

USB, Wi-Fi or Ethernet connections can fail.

## Benefit

```text
Raw data remains available even if dashboard or network fails.
```

## Data policy

```text
Raw waveform data is stored first.
Processed/filtered data is generated later.
```

This prevents loss of scientific information caused by early filtering.

---

# 11. Environmental Sensor Fusion

## Decision

Measure temperature, humidity, vibration and wind together with pressure.

## Why

A pressure waveform can be caused by:

- Real atmospheric infrasound
- Wind turbulence
- Enclosure vibration
- Thermal drift
- Moisture blockage
- Power noise

## Benefit

```text
Environmental channels help determine whether a pressure waveform
is likely a real event or environmental/electronic interference.
```

---

# 12. Temperature Compensation

## Decision

Use two TMP117 temperature sensors.

```text
TMP117 #1 = sensor PCB / analog front end
TMP117 #2 = backing chamber
```

## Why

Temperature affects:

- Sensor offset
- Sensor sensitivity
- Capillary resistance
- Backing chamber pressure
- Analog amplifier offset
- Voltage reference

## Compensation model

```text
P_corrected = [P_measured - Offset(T)] / Sensitivity(T)
```

---

# 13. USB Dashboard Instead of Embedded Graphical UI

## Decision

Use USB CDC serial output to a laptop Python dashboard.

## Why

A large display near sensitive analog electronics can add switching noise and consume power.

## Benefit

```text
The STM32 focuses on reliable acquisition and logging.
The laptop handles plots, spectrograms and presentation-quality visuals.
```

---

# 14. Force Feedback as Future Innovation Module

## Decision

Treat force feedback as a Phase 2 innovation instead of the first prototype requirement.

## Why

Force feedback requires:

- Precision diaphragm/bellows
- Position sensing
- Voice coil or piezo actuator
- Current driver
- PID control tuning
- Force calibration
- Mechanical alignment

## Benefit

```text
The team can first demonstrate a validated passive differential
microbarometer and later add force-feedback self-calibration.
```

## Future architecture

```text
Pressure → Diaphragm displacement → Position sensor
         → PID controller → Actuator restoring force
         → Feedback current → Equivalent pressure
```

---

# 15. Two-Stage Development Strategy

| Stage | Architecture | Main goal |
|---|---|---|
| Version 1 | Differential MEMS + pneumatic filter + 24-bit ADC | Working, calibrated infrasound prototype |
| Version 2 | Add force-feedback and self-calibration | Improve dynamic range, diagnostics and innovation |
| Version 3 | GPS synchronized multi-node array | Direction finding and source localization |

---

# 16. Final Design Decision Summary

```text
Pressure sensing:
Low-range differential MEMS sensor

Pneumatic filter:
Backing chamber + interchangeable capillary

Wind reduction:
Four-arm pipe rosette, upgradeable to eight arms

ADC:
ADS131M04 24-bit simultaneous-sampling ADC

Controller:
STM32H743

Timing:
GPS PPS + DS3231 RTC

Logging:
High-endurance microSD

Visualization:
USB-connected Python dashboard

Processing:
CMSIS-DSP on STM32 + Python/ObsPy offline analysis

Innovation:
Environmental sensor fusion + future force-feedback path
```
