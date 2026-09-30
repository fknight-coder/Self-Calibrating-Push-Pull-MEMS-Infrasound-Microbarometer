# System Architecture

## ApeX-InfraSense: High-Sensitivity Infrasound Microbarometer

This document describes the complete system architecture of the ApeX-InfraSense project.

The proposed system is a differential atmospheric microbarometer designed to detect and record infrasonic pressure fluctuations in the frequency range:

```text
0.01 Hz to 20 Hz
```

The architecture combines:

- Differential atmospheric pressure sensing
- Pneumatic long-period pressure equalization
- Wind-noise-reduction pipe rosette
- Low-noise analog signal conditioning
- 24-bit delta-sigma digitization
- STM32-based embedded acquisition and digital signal processing
- Temperature, vibration and wind monitoring
- GPS-synchronized timestamps
- microSD waveform storage
- Real-time laptop/dashboard visualization
- Future force-feedback self-calibration path

---

## 1. System-Level Architecture

```text
                     ┌──────────────────────────────┐
                     │ Atmospheric Infrasound Source │
                     │ 0.01–20 Hz Pressure Wave      │
                     └──────────────┬───────────────┘
                                    │
                                    ▼
                 ┌──────────────────────────────────┐
                 │ Wind-Noise-Reduction Pipe Rosette│
                 │ Multi-arm spatial pressure filter │
                 └──────────────┬───────────────────┘
                                │
                                ▼
              ┌─────────────────────────────────────────┐
              │ Inlet Protection Layer                   │
              │ Mesh + PTFE membrane + moisture trap     │
              └──────────────┬──────────────────────────┘
                             │
                             ▼
       ┌──────────────────────────────────────────────────────┐
       │ Differential Pressure Sensing Head                    │
       │                                                      │
       │  P1: Atmospheric pressure from wind rosette          │
       │  P2: Pneumatic backing-chamber reference pressure    │
       │                                                      │
       │  Measured signal: Delta-P = P1 - P2                  │
       └──────────────┬───────────────────────┬──────────────┘
                      │                       │
                      │                       │
                      ▼                       ▼
       ┌─────────────────────────┐   ┌─────────────────────────┐
       │ Low-Range Differential  │   │ Backing Chamber +        │
       │ MEMS / Bellows Sensor   │   │ Capillary Equalization   │
       └──────────────┬──────────┘   └─────────────────────────┘
                      │
                      ▼
       ┌──────────────────────────────────────────────────────┐
       │ Low-Noise Analog Front End                            │
       │ Buffer + stable reference + anti-alias LPF            │
       │ Recommended analog bandwidth: 0–25 Hz                 │
       └──────────────┬───────────────────────────────────────┘
                      │
                      ▼
       ┌──────────────────────────────────────────────────────┐
       │ ADS131M04 24-bit Delta-Sigma ADC                      │
       │ CH0: Main pressure | CH1: Reference pressure          │
       │ CH2: Vibration    | CH3: Calibration/diagnostic      │
       └──────────────┬───────────────────────────────────────┘
                      │ SPI + DRDY + DMA
                      ▼
       ┌──────────────────────────────────────────────────────┐
       │ STM32H743 Embedded Controller                         │
       │ Acquisition + DSP + time sync + logging + control     │
       └───┬───────────────┬─────────────────┬────────────────┘
           │               │                 │
           ▼               ▼                 ▼
  ┌────────────────┐ ┌──────────────┐ ┌───────────────────────┐
  │ GPS PPS + RTC  │ │ microSD Card │ │ USB / Ethernet Output │
  │ UTC Time Sync  │ │ Raw Waveform │ │ Laptop/Web Dashboard  │
  └────────────────┘ └──────────────┘ └───────────────────────┘
           │
           ▼
  ┌───────────────────────────────────────────────────────────┐
  │ Environmental Sensor Fusion                                │
  │ Temperature + humidity + wind + vibration + battery health │
  └───────────────────────────────────────────────────────────┘
```

---

## 2. Functional Data Flow

```text
Atmospheric pressure wave
        │
        ▼
Wind-rosette spatial filter
        │
        ▼
Differential sensor measures P1 - P2
        │
        ▼
Backing chamber removes slow weather-pressure drift
        │
        ▼
Analog front end filters and conditions weak sensor waveform
        │
        ▼
24-bit ADC converts analog waveform to digital samples
        │
        ▼
STM32 performs calibration and temperature compensation
        │
        ▼
STM32 computes filtering, RMS, FFT, PSD and event trigger
        │
        ├── Save raw data to microSD
        ├── Send live data to laptop through USB
        ├── Display sensor status on OLED
        └── Generate event/fault alert
```

---

## 3. Pressure-Sensing Architecture

The microbarometer does not measure only absolute atmospheric pressure. It measures the small pressure difference between an atmospheric inlet and a controlled reference path.

```text
P1 = Wind-filtered atmospheric pressure
P2 = Filtered reference pressure inside backing chamber

Delta-P = P1 - P2
```

### P1: Atmospheric pressure input

The P1 pressure port receives atmospheric pressure from the pipe rosette.

```text
Atmosphere
    │
    ▼
Pipe-rosette inlets
    │
    ▼
Central manifold
    │
    ▼
PTFE membrane + dust mesh + moisture trap
    │
    ▼
Sensor P1 port
```

### P2: Reference pressure input

The P2 pressure port connects to the backing chamber and capillary system.

```text
Sensor P2 port
    │
    ▼
Rigid backing chamber
    │
    ▼
Fine capillary restriction
    │
    ▼
Slow pressure equalization path
```

### Purpose of differential sensing

| Atmospheric effect | P1 response | P2 response | Differential output |
|---|---|---|---|
| Fast infrasound wave | Changes quickly | Changes slowly | Detected |
| Slow weather pressure change | Changes slowly | Gradually equalizes | Rejected |
| Local wind turbulence | Reduced by rosette | Reference remains stable | Reduced |
| Temperature drift | May change sensor offset | Corrected digitally | Compensated |
| Strong pressure event | Large transient | Delayed reference response | Recorded within sensor range |

---

## 4. Pneumatic Equalization Architecture

The backing chamber and capillary act as a pneumatic high-pass filter.

```text
Fast pressure variation:
P1 changes rapidly
P2 cannot equalize rapidly
Delta-P appears across sensor diaphragm

Slow pressure variation:
P1 changes slowly
P2 gradually equalizes through capillary
Delta-P approaches zero
```

### Approximate pneumatic model

```text
fc = 1 / (2 × pi × R × C)
```

Where:

| Parameter | Meaning |
|---|---|
| `fc` | Pneumatic cutoff frequency |
| `R` | Pneumatic resistance of capillary tube |
| `C` | Pneumatic compliance of backing chamber |
| `P1` | Atmospheric pressure input |
| `P2` | Reference pressure |
| `Delta-P` | Differential infrasound pressure |

### Initial pneumatic design targets

| Parameter | Initial target |
|---|---:|
| Measurement band | 0.01–20 Hz |
| Lower cutoff frequency | Approximately 0.01 Hz |
| Backing chamber volume | 1–5 cm³ |
| Capillary internal diameter | 25–100 µm |
| Capillary length | 20–150 mm |
| Capillary design | Replaceable/interchangeable cartridges |
| Chamber material | Aluminium, brass, steel or rigid acrylic |
| Required property | Airtight and mechanically rigid |

> The final capillary and chamber dimensions must be selected experimentally after measuring the complete frequency response of the sensor.

---

## 5. Wind-Noise-Reduction Architecture

Wind turbulence is one of the largest sources of unwanted pressure noise in outdoor infrasound measurements.

The system uses a multi-arm pneumatic pipe rosette.

```text
      Protected inlet
            │
            ▼
       ── Pipe Arm ──
            │
Protected ──┼── Pipe Arm ──┐
Inlet       │              │
            ├── Pipe Arm ──┼── Central Manifold ── Sensor P1
            │              │
Protected ──┼── Pipe Arm ──┘
Inlet       │
       ── Pipe Arm ──
            │
      Protected inlet
```

### Rosette design

| Feature | Initial prototype | Final upgrade |
|---|---:|---:|
| Number of arms | 4 | 8 |
| Arm length | 1–2 m | 3–5 m |
| Pipe material | PE/PVC/PTFE tubing | UV-resistant PE/PTFE tubing |
| Pipe geometry | Equal length | Equal length and equal inlet layout |
| Inlet protection | Mesh + PTFE membrane | Mesh + PTFE membrane + replaceable filter |
| Main purpose | Basic wind reduction | Better outdoor turbulence reduction |

### Why it works

```text
Wind turbulence:
Small-scale and uncorrelated between different inlets
→ Reduced when inlet pressure is spatially averaged

Infrasound:
Long wavelength and coherent across the full rosette
→ Preserved after combining multiple inlet pressures
```

---

## 6. Analog Front-End Architecture

The analog front end prepares the weak sensor output for the ADC without adding significant low-frequency noise, offset, or drift.

```text
Pressure Sensor Output
        │
        ▼
Precision Low-Offset Buffer
        │
        ▼
Reference / Offset Conditioning
        │
        ▼
2nd or 4th Order Active Low-Pass Filter
Cutoff: approximately 20–25 Hz
        │
        ▼
ADC Input Driver
        │
        ▼
ADS131M04 Differential Input
```

### Analog front-end requirements

| Requirement | Technical approach |
|---|---|
| Low offset drift | Precision low-drift op-amp |
| Low 1/f noise | Low-noise precision analog components |
| Stable ADC reference | Low-drift voltage reference |
| Anti-aliasing | 20–25 Hz active low-pass filter |
| Power-supply noise reduction | Separate analog low-noise LDO |
| Digital noise isolation | Separate analog/digital regions |
| Signal integrity | Short traces and shielded cable |
| Input protection | Series resistors and low-leakage protection |
| PCB quality | Four-layer PCB preferred |

### Power architecture

```text
12 V Input / Battery
        │
        ├── Digital Buck Regulator
        │        │
        │        └── STM32 + GPS + microSD + display
        │
        └── Low-Noise Analog LDO
                 │
                 ├── Differential pressure sensor
                 ├── Precision op-amps
                 ├── Voltage reference
                 └── ADC analog supply
```

---

## 7. ADC and Data-Acquisition Architecture

### ADC selection

The proposed primary ADC is:

```text
ADS131M04
4-channel simultaneous-sampling
24-bit delta-sigma ADC
SPI communication interface
```

### ADC channel map

| ADC channel | Connected signal | Purpose |
|---|---|---|
| CH0 | Main differential pressure signal | Primary infrasound waveform |
| CH1 | Secondary pressure/reference channel | Redundancy, comparison or second pneumatic path |
| CH2 | Accelerometer analog output | Identify vibration artifacts |
| CH3 | Calibration/current-monitor signal | Self-test and system diagnostics |

### Sampling architecture

```text
ADS131M04 conversion completes
        │
        ▼
DRDY signal goes active
        │
        ▼
STM32 external interrupt occurs
        │
        ▼
STM32 SPI + DMA reads ADC data frame
        │
        ▼
24-bit data converted to signed 32-bit variables
        │
        ▼
Samples placed into circular buffer
        │
        ▼
Data logging and DSP tasks process buffered data
```

### Sampling configuration

| Parameter | Recommended initial value |
|---|---:|
| ADC sample rate | 250–1000 SPS |
| Stored waveform rate | 100 SPS |
| Maximum required signal frequency | 20 Hz |
| Nyquist frequency at 100 SPS | 50 Hz |
| Analog low-pass filter | 20–25 Hz |
| Digital output format | Signed 32-bit pressure data |
| ADC transfer mode | SPI + DMA |
| Timing source | GPS PPS + RTC backup |

---

## 8. STM32 Embedded Architecture

The STM32H743 is the central embedded controller.

```text
                 ┌────────────────────────────┐
                 │         STM32H743          │
                 └────────────────────────────┘
                    │      │      │      │
                    │      │      │      │
                    ▼      ▼      ▼      ▼
                  ADC     GPS    SD     USB
                  SPI     PPS   SDMMC   CDC
                    │
                    ▼
               CMSIS-DSP
                    │
                    ▼
        Event Detection and Health Monitoring
```

### Main STM32 functions

| Embedded function | Description |
|---|---|
| ADC acquisition | Reads ADS131M04 samples using SPI + DMA |
| Sample buffering | Uses circular/double buffers in RAM |
| GPS synchronization | Captures PPS timing and UTC data |
| RTC backup | Maintains time when GPS is unavailable |
| Data logging | Saves waveform blocks to microSD |
| Pressure conversion | Converts ADC counts to pressure in Pa |
| Temperature compensation | Corrects temperature-related offset/sensitivity |
| Digital filtering | Applies analysis band-pass and low-pass filters |
| RMS calculation | Calculates signal/noise amplitude |
| FFT and PSD | Generates spectrum and noise-floor information |
| Event triggering | Uses STA/LTA and amplitude thresholds |
| Environmental fusion | Uses wind, vibration and temperature data |
| USB communication | Streams live waveform to laptop |
| Display control | Updates OLED/TFT local status display |
| Calibration control | Controls actuator, valve or self-test signal |
| System health | Checks ADC, SD, GPS, supply and sensor status |

---

## 9. Environmental Sensor Architecture

```text
Temperature Sensor #1 → Pressure sensor / analog PCB
Temperature Sensor #2 → Backing chamber
Humidity Sensor       → Enclosure moisture condition
Absolute Barometer    → Atmospheric trend diagnostic
Accelerometer         → Enclosure vibration reference
Anemometer            → Wind-speed reference
GPS Receiver          → UTC timing and PPS
Power Monitor         → Battery / supply health
```

### Environmental sensor roles

| Sensor | Typical device | Purpose |
|---|---|---|
| Precision temperature | TMP117 | Sensor and chamber temperature correction |
| Humidity | SHT45 | Condensation-risk monitoring |
| Absolute pressure | BMP390 | Weather-pressure trend and diagnostics |
| Accelerometer | ADXL355 | Mechanical vibration identification |
| Wind sensor | Cup/ultrasonic anemometer | Wind-noise correlation |
| GPS | u-blox module with PPS | Accurate timestamp synchronization |
| RTC | DS3231 | Backup timing |
| Power monitor | INA226 | Battery voltage/current measurement |

---

## 10. Signal Processing Architecture

```text
Raw ADC Samples
        │
        ▼
ADC Code to Voltage Conversion
        │
        ▼
Voltage to Pressure Conversion
        │
        ▼
Temperature Offset Compensation
        │
        ▼
Sensitivity Compensation
        │
        ▼
0.01–20 Hz Analysis Filter
        │
        ├── RMS / Peak Amplitude
        ├── FFT
        ├── Power Spectral Density
        ├── Spectrogram Features
        ├── STA/LTA Trigger
        ├── Wind Correlation
        └── Vibration Correlation
        │
        ▼
Validated Infrasound Waveform + Event Metadata
```

### Core calculations

```text
Differential pressure:
Delta-P = P1 - P2

Temperature correction:
P_corrected = [P_measured - Offset(T)] / Sensitivity(T)

RMS pressure:
P_RMS = sqrt(sum(P[n]^2) / N)

Event trigger:
STA/LTA = short-term average energy / long-term average energy
```

### Event-validation logic

| Observation | System interpretation |
|---|---|
| High pressure energy + low wind + low vibration | Possible real infrasound event |
| High pressure energy + high wind correlation | Likely wind noise |
| High pressure energy + high vibration correlation | Likely mechanical artifact |
| High coherence across pressure channels | Higher event confidence |
| GPS synchronized signal at multiple stations | Candidate distant event |
| Sensor offset increase + moisture rise | Possible condensation/leakage issue |

---

## 11. Data Storage and Communication Architecture

```text
STM32H743
   │
   ├── microSD Card
   │      ├── Raw waveform data
   │      ├── Processed waveform data
   │      ├── Event waveform data
   │      ├── Environmental metadata
   │      └── System-health logs
   │
   ├── USB CDC Serial
   │      └── Laptop real-time waveform dashboard
   │
   ├── Ethernet
   │      └── Optional local network/web dashboard
   │
   └── Optional LoRaWAN / 4G / NB-IoT
          └── Remote event alerts and field deployment
```

### Data types

| Data type | Example format | Purpose |
|---|---|---|
| Raw waveform | `.bin` / `.dat` | Preserve original ADC output |
| Processed waveform | `.csv` / `.mseed` | Analysis and waveform sharing |
| Environmental data | `.csv` / `.json` | Temperature, wind, vibration and humidity |
| Event metadata | `.json` | Trigger time, frequency, amplitude, confidence |
| Calibration coefficients | `.json` | Offset, sensitivity and thermal coefficients |
| System health | `.csv` / `.json` | GPS, SD, ADC and power status |

---

## 12. Calibration Architecture

```text
Laptop / Signal Generator
          │
          ▼
Speaker / Piston / Motorized Syringe
          │
          ▼
Rigid Sealed Calibration Chamber
          │
          ├── Reference Pressure Sensor
          ├── Kestrel-InfraSense Sensor Under Test
          └── Chamber Temperature Sensor
```

### Calibration sequence

```text
1. Apply a known pressure waveform
2. Measure reference pressure
3. Record sensor-under-test output
4. Compare amplitude and phase
5. Calculate sensitivity
6. Store calibration coefficients
7. Repeat across frequency and temperature
```

### Calibration frequencies

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

### Calibration outputs

| Measurement | Output |
|---|---|
| Frequency response | Amplitude vs frequency graph |
| Phase response | Phase difference vs frequency graph |
| Sensitivity | Pa to ADC counts / V per Pa |
| Linearity | Reference pressure vs measured pressure |
| Noise floor | RMS noise and PSD graph |
| Temperature drift | Offset vs temperature graph |
| Stability | Offset/noise trend over time |
| Wind reduction | Bare inlet vs rosette noise PSD |

---

## 13. Optional Force-Feedback Architecture

> This is an advanced future implementation path. It should be presented as a research/innovation module until physically built and calibrated.

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
Position Error Signal
        │
        ▼
STM32 PID Controller
        │
        ▼
DAC + Bidirectional Current Driver
        │
        ▼
Voice-Coil / Magnetic Actuator
        │
        ▼
Restoring Force Maintains Near-Null Position
        │
        ▼
Feedback Current Used as Pressure Estimate
```

### Force-feedback concept

```text
Feedback force:
F_feedback = Kf × I_feedback

Equivalent pressure:
Delta-P = F_feedback / A

Therefore:
Delta-P = (Kf × I_feedback) / A
```

Where:

| Variable | Meaning |
|---|---|
| `F_feedback` | Restoring force generated by actuator |
| `Kf` | Actuator force constant |
| `I_feedback` | Feedback current |
| `A` | Effective diaphragm/bellows area |
| `Delta-P` | Measured pressure fluctuation |

### Benefits of force feedback

- Keeps the sensing element near its linear null position
- Extends usable dynamic range
- Reduces mechanical nonlinearity and hysteresis
- Enables electrical self-test through known current injection
- Supports actuator-current and null-position health diagnostics
- Provides a path toward professional-grade active microbarometer design

---

## 14. System States

| State | Description |
|---|---|
| Boot | MCU starts and checks power, ADC, SD and sensors |
| Initialization | GPS, RTC, ADC and environmental sensors initialize |
| Warm-up | Analog reference and pressure sensor stabilize |
| Acquisition | Continuous ADC sampling and circular buffering |
| Logging | Waveforms and metadata are stored to microSD |
| Monitoring | Dashboard and OLED show live status |
| Event mode | Triggered waveform and metadata are saved |
| Calibration mode | Known test signal is applied and response is measured |
| Fault mode | Sensor, SD, GPS, ADC or power fault is reported |
| Maintenance mode | User checks chamber, capillary, filters and rosette |

---

## 15. Performance Verification Plan

| Requirement | Verification method |
|---|---|
| 0.01–20 Hz detection | Frequency sweep in sealed calibration chamber |
| Pressure sensitivity | Known input pressure versus output calibration |
| Frequency response | Bode magnitude and phase plots |
| Noise floor | Equalized-port recording and PSD/RMS calculation |
| Long-term stability | 24-hour, 7-day and extended drift measurement |
| Temperature compensation | Multi-temperature offset/sensitivity test |
| Wind-noise reduction | Bare inlet vs rosette outdoor PSD comparison |
| Data continuity | Long-duration microSD logging test |
| Time accuracy | GPS PPS timestamp verification |
| Event detection | Controlled pressure pulses and signal injection |
| Force-feedback path | Null control and current-to-pressure calibration |

---

## 16. Architecture Design Principles

```text
1. Measure differential pressure, not only absolute pressure.
2. Remove slow atmospheric drift pneumatically before digital processing.
3. Reduce wind noise mechanically before increasing electronic gain.
4. Keep analog circuitry isolated from digital switching noise.
5. Store raw waveform data before applying irreversible processing.
6. Use calibration data to convert every ADC sample into pressure.
7. Measure temperature, vibration and wind with the pressure waveform.
8. Treat force feedback as an advanced controlled module.
9. Validate every performance claim through measured test results.
10. Design the node so it can later become part of a GPS-synchronized array.
```

---

## 17. References

1. Infra-NMT differential MEMS microbarometer and mechanical filtering methodology.
2. CTBTO infrasound monitoring and wind-noise-reduction pipe-array concepts.
3. ADS131M04 datasheet and evaluation-board documentation.
4. STM32H743, STM32CubeIDE, FreeRTOS, FatFs and CMSIS-DSP documentation.
5. ObsPy waveform-analysis and MiniSEED documentation.
6. Infrasound calibration methods using sealed cavities, reference sensors and piston-based excitation.
7. Research on force-feedback/closed-loop infrasound and pressure-sensing architectures.
