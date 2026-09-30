SIH26058-Team-Kestrel
# Kestrel-InfraSense  
## High-Sensitivity Differential Microbarometer for Infrasound Detection

> **Smart India Hackathon Project — Team Kestrel**  
> A low-noise, STM32-based atmospheric microbarometer for detecting and analyzing infrasonic pressure fluctuations in the **0.01–20 Hz** frequency range.

![Project Status](https://img.shields.io/badge/Status-In%20Development-orange)
![Platform](https://img.shields.io/badge/Platform-STM32H743-blue)
![ADC](https://img.shields.io/badge/ADC-24--bit%20Delta--Sigma-green)
![Signal Range](https://img.shields.io/badge/Frequency-0.01--20%20Hz-purple)
![License](https://img.shields.io/badge/License-MIT-lightgrey)

---

## Problem Statement

Infrasound consists of low-frequency atmospheric pressure waves below the lower limit of human hearing, typically below **20 Hz**. These signals can travel long distances through the atmosphere and can originate from:

- Industrial explosions and mining blasts
- Volcanic eruptions
- Severe weather systems and thunderstorms
- Meteors and fireballs
- Rocket launches
- Ocean microbaroms
- Atmospheric disturbances
- Security-related energetic events

The main engineering challenge is that useful infrasonic signals may be extremely small compared with normal atmospheric pressure and environmental noise.

This project addresses the design and development of a high-sensitivity atmospheric microbarometer capable of measuring pressure fluctuations from:

\[
0.01Hz to 20 Hz
\]

while reducing the effects of:

- Slow weather-pressure drift
- Wind turbulence
- Temperature-induced sensor drift
- Moisture and dust ingress
- Mechanical vibration
- Electronic noise
- ADC quantization noise
- Long-term instability

---

## Proposed Solution

**Kestrel-InfraSense** is a differential infrasound sensor platform built around:

- A low-range differential MEMS pressure sensor or bellows-based transducer
- Pneumatic long-period equalization using a backing chamber and capillary path
- Multi-arm wind-noise-reduction pipe rosette
- Low-noise analog signal-conditioning circuit
- 24-bit simultaneous-sampling ADC
- STM32H743 embedded controller
- GPS PPS-based timestamp synchronization
- High-endurance microSD waveform storage
- Temperature, humidity, wind and vibration monitoring
- Real-time USB/Ethernet waveform visualization
- Calibration, frequency-response and noise-floor measurement tools
- Future force-feedback self-calibration and array-processing capability

---

## Key Objectives

| Objective | Target |
|---|---:|
| Infrasound frequency range | 0.01–20 Hz |
| Pressure measurement principle | Differential pressure sensing |
| Recommended sensor range | ±125 Pa to ±250 Pa |
| Main digitizer | 24-bit delta-sigma ADC |
| Data-acquisition controller | STM32H743 |
| Stored waveform rate | 100 samples/s |
| Internal acquisition rate | 250–1000 samples/s |
| Wind-noise reduction | Four/eight-arm pipe rosette |
| Time synchronization | GPS PPS + RTC backup |
| Real-time display | USB/Ethernet laptop dashboard |
| Storage | High-endurance microSD card |
| Calibration | Pressure chamber + reference sensor |
| Key evaluation metrics | Frequency response, sensitivity, noise floor, drift, stability |

> **Note:** All frequency response, sensitivity, pressure resolution, and noise figures are engineering targets until validated by laboratory calibration and field testing.

---

## System Architecture

```text
                Atmospheric Infrasound Wave
                          │
                          ▼
          Multi-Arm Wind-Noise-Reduction Rosette
                          │
                          ▼
       PTFE Filter + Mesh + Moisture Protection
                          │
                          ▼
          Differential Pressure Sensor / Transducer
               ┌──────────┴──────────┐
               │                     │
               ▼                     ▼
      P1: Atmospheric Input     P2: Reference Path
                                      │
                                      ▼
                       Backing Chamber + Capillary
                       Long-Period Pressure Equalization
                                      │
                                      ▼
                         Low-Noise Analog Front End
                      Buffer + Reference + 25 Hz Filter
                                      │
                                      ▼
                  ADS131M04 24-bit Delta-Sigma ADC
                                      │
                         SPI + DRDY + DMA Transfer
                                      │
                                      ▼
                              STM32H743 Controller
               ┌──────────────┼───────────────┐
               ▼              ▼               ▼
         GPS PPS Timing   microSD Logging   USB/Ethernet
               │              │               │
               └──────────────┴───────────────┘
                                      │
                                      ▼
          Real-Time Waveform + FFT + Spectrogram + Event Detection
```

---

## Working Principle

### Differential pressure measurement

The sensor measures the difference between atmospheric pressure at the inlet and a filtered reference pressure:

\[
\Delta P = P_1 - P_2
\]

Where:

- \(P_1\) = wind-filtered atmospheric pressure
- \(P_2\) = backing-chamber reference pressure
- \(\Delta P\) = desired infrasound pressure waveform

### Pneumatic high-pass behavior

The backing chamber and capillary create a pneumatic high-pass response:

```text
Fast infrasonic pressure change:
P1 changes rapidly → P2 changes slowly → ΔP is detected

Slow weather-pressure change:
P1 changes slowly → P2 equalizes through capillary → drift is removed
```

The approximate pneumatic cutoff is modeled as:

\[
f_c = \frac{1}{2\pi RC}
\]

where:

- \(R\) = pneumatic resistance of the capillary
- \(C\) = pneumatic compliance of the backing volume

The design target is:

\[
f_c \approx 0.01\ \text{Hz}
\]

---

## Innovation and USP

### 1. Tunable pneumatic filter

Instead of a fixed pneumatic filter, the system uses interchangeable capillary cartridges and backing-volume configurations.

This enables experimental tuning of the low-frequency response near:

\[
0.01\ \text{Hz}
\]

### 2. Adaptive wind-noise validation

The system combines:

```text
Pressure waveform
+ Wind speed
+ Vibration data
+ Temperature trend
= Event confidence score
```

This helps distinguish:

- Genuine infrasonic events
- Wind-generated turbulence
- Mechanical vibration
- Sensor handling artifacts
- Pipe blockage or condensation problems

### 3. Multi-channel synchronous digitization

The 24-bit ADC can acquire several channels simultaneously:

| ADC Channel | Signal |
|---|---|
| CH0 | Main differential pressure waveform |
| CH1 | Secondary pressure/reference channel |
| CH2 | Vibration/accelerometer reference |
| CH3 | Calibration or diagnostic channel |

### 4. Embedded standalone architecture

The STM32H743 handles:

- ADC acquisition
- SPI-DMA transfer
- Temperature compensation
- GPS timestamps
- microSD logging
- FFT and spectral features
- RMS/noise computation
- Event triggering
- USB/Ethernet communication

No Raspberry Pi is required for core acquisition and storage.

### 5. Future force-feedback path

A future version can include a voice-coil or piezoelectric force-feedback mechanism.

```text
Pressure → Diaphragm displacement → Position error
         → PID controller → Actuator restoring force
         → Feedback current → Pressure estimate
```

\[
\Delta P = \frac{K_f \times I_{feedback}}{A}
\]

Where:

- \(K_f\) = actuator force constant
- \(I_{feedback}\) = feedback current
- \(A\) = effective diaphragm/bellows area

This can improve linearity, dynamic range, self-calibration capability, and health monitoring.

---

## Hardware Stack

| Layer | Technology / Component |
|---|---|
| Main pressure sensing | Low-range differential MEMS pressure sensor / precision bellows |
| Wind-noise reduction | Four/eight-arm pipe rosette |
| Pneumatic filter | Rigid backing chamber + micro-capillary |
| Analog front end | OPA188/OPA189 low-drift precision op-amp |
| Analog filtering | 20–25 Hz anti-alias low-pass filter |
| Voltage reference | REF5025 low-drift precision reference |
| Analog supply | TPS7A20 low-noise LDO |
| Main ADC | ADS131M04 24-bit delta-sigma ADC |
| Main MCU | STM32H743ZI2 / STM32H743ZIT6 |
| Temperature sensing | TMP117 |
| Humidity sensing | SHT45 |
| Absolute pressure | BMP390 |
| Vibration sensing | ADXL355 |
| Wind sensing | Cup or ultrasonic anemometer |
| Time synchronization | u-blox GPS module with PPS |
| Backup RTC | DS3231 |
| Storage | High-endurance microSD using SDMMC |
| Demonstration output | USB CDC serial / Ethernet |
| Optional remote communication | LoRaWAN, 4G, NB-IoT, Wi-Fi module |
| Calibration system | Sealed chamber + piston/speaker + reference sensor |

---

## Software Stack

| Purpose | Technology |
|---|---|
| STM32 firmware development | STM32CubeIDE |
| MCU peripheral configuration | STM32CubeMX |
| Embedded language | C/C++ |
| Real-time scheduling | FreeRTOS |
| DSP filtering and FFT | CMSIS-DSP |
| SD-card file system | FatFs |
| Desktop/laptop dashboard | Python |
| Serial communication | PySerial |
| Signal analysis | NumPy + SciPy |
| Live visualization | PyQtGraph / Plotly / Matplotlib |
| Infrasound waveform analysis | ObsPy |
| PCB design | KiCad |
| Mechanical/CAD design | Fusion 360 / SolidWorks / FreeCAD |
| Version control | Git + GitHub |

---

## Repository Structure

```text
SIH26058-Team-Kestrel/
│
├── README.md
├── LICENSE
├── CONTRIBUTING.md
├── .gitignore
│
├── docs/
│   ├── problem-statement.md
│   ├── system-architecture.md
│   ├── technical-flow.md
│   ├── innovation-and-usp.md
│   ├── calibration-methodology.md
│   ├── test-plan.md
│   ├── references.md
│   ├── diagrams/
│   ├── images/
│   └── presentations/
│
├── hardware/
│   ├── bom/
│   ├── mechanical/
│   │   ├── backing-chamber/
│   │   ├── capillary-cartridges/
│   │   ├── wind-rosette/
│   │   ├── calibration-chamber/
│   │   └── enclosure/
│   │
│   ├── electronics/
│   │   ├── analog-front-end/
│   │   ├── adc-board/
│   │   ├── stm32-controller/
│   │   ├── power-supply/
│   │   └── force-feedback/
│   │
│   ├── kicad/
│   ├── schematics/
│   └── datasheets/
│
├── firmware/
│   ├── stm32h743/
│   ├── libraries/
│   └── tools/
│
├── software/
│   ├── dashboard/
│   ├── analysis/
│   ├── simulation/
│   └── notebooks/
│
├── data/
│   ├── sample_data/
│   ├── processed_data/
│   └── metadata/
│
├── tests/
│   ├── hardware_tests/
│   ├── calibration_tests/
│   ├── stability_tests/
│   └── results/
│
├── config/
│   ├── adc_config.json
│   ├── sensor_config.json
│   ├── calibration_config.json
│   └── event_detection_config.json
│
├── scripts/
│   ├── setup_environment.sh
│   ├── run_dashboard.sh
│   ├── convert_binary_to_csv.py
│   ├── convert_binary_to_mseed.py
│   └── generate_test_signal.py
│
├── .github/
│   ├── workflows/
│   ├── ISSUE_TEMPLATE/
│   └── pull_request_template.md
│
└── assets/
    ├── logo/
    ├── screenshots/
    ├── demo-gifs/
    └── posters/
```

---

## Firmware Flow

```text
Power ON
    │
    ▼
Initialize STM32, ADC, microSD, GPS, RTC and sensors
    │
    ▼
ADS131M04 produces DRDY interrupt
    │
    ▼
STM32 reads 24-bit ADC frame using SPI + DMA
    │
    ▼
ADC samples stored in circular RAM buffer
    │
    ▼
Convert ADC counts into pressure values
    │
    ▼
Apply temperature and sensitivity compensation
    │
    ▼
Store raw waveform to microSD in blocks
    │
    ▼
Apply DSP filters, RMS, FFT, PSD and STA/LTA trigger
    │
    ▼
Evaluate wind and vibration correlation
    │
    ▼
Transmit live waveform to dashboard through USB/Ethernet
```

---

## Calibration Methodology

The sensor must be calibrated using a sealed pressure chamber.

```text
Signal Generator
      │
      ▼
Speaker / Piston / Motorized Syringe
      │
      ▼
Sealed Calibration Chamber
      │
      ├── Reference Pressure Sensor
      └── Kestrel-InfraSense Sensor Under Test
```

### Frequency-response test points

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

### Measurements

| Metric | Method |
|---|---|
| Frequency response | Compare sensor amplitude with calibrated reference |
| Phase response | Measure phase difference between reference and test sensor |
| Sensitivity | Output voltage/ADC counts per pascal |
| Linearity | Apply several known pressure amplitudes |
| Noise floor | Equalize both ports and calculate RMS/PSD |
| Temperature drift | Repeat tests across controlled temperature points |
| Long-term stability | Record offset and RMS noise for 24 hours to 30 days |
| Wind-noise reduction | Compare bare inlet versus pipe rosette PSD |

### Core equations

\[
Sensitivity(f)=\frac{V_{out}(f)}{P_{reference}(f)}
\]

\[
P_{RMS}=
\sqrt{
\frac{1}{N}\sum_{n=1}^{N}P_n^2
}
\]

\[
Drift=
\frac{P_{offset,end}-P_{offset,start}}{\Delta t}
\]

---

## Real-Time Dashboard

The dashboard displays:

```text
-  Live pressure waveform in Pa
-  Raw and filtered waveform
-  RMS pressure/noise level
-  FFT frequency spectrum
-  Spectrogram
-  Dominant frequency
-  Temperature and humidity
-  Wind speed
-  Vibration level
-  GPS lock and UTC time
-  SD-card recording state
-  Battery/power health
-  Event trigger status
-  Calibration version
```

---

## SIH Evaluation Mapping

| Evaluation Requirement | Kestrel-InfraSense Evidence |
|---|---|
| Detection of low-frequency signals | Live waveform from 0.01–20 Hz calibration source |
| Laboratory frequency characterization | Bode magnitude and phase-response plots |
| Noise-floor measurement | RMS noise values and PSD graphs |
| Sensitivity estimation | Pressure vs output/ADC calibration graph |
| Stability testing | 24-hour, 7-day and long-duration drift plots |
| Differential sensing | P1/P2 pneumatic path demonstration |
| Pressure equalization | Step response and pneumatic cutoff graph |
| Wind-noise reduction | Bare inlet vs pipe-rosette comparison |
| Temperature compensation | Before/after thermal correction results |
| Functional DAQ system | STM32, ADC, GPS, SD storage and dashboard demo |
| Real-time display | USB/Ethernet live waveform, FFT and spectrogram |

---

## Development Roadmap

- [x] Define project architecture and technical requirements
- [x] Select STM32-based data-acquisition architecture
- [x] Select 24-bit ADC approach
- [x] Define wind-rosette and pneumatic filter concept
- [ ] Procure differential sensor and ADC evaluation board
- [ ] Build backing chamber and interchangeable capillary system
- [ ] Implement ADS131M04 SPI + DMA firmware driver
- [ ] Add GPS PPS timestamp synchronization
- [ ] Add microSD waveform logging
- [ ] Build Python real-time waveform dashboard
- [ ] Build calibration chamber
- [ ] Measure frequency response from 0.01–20 Hz
- [ ] Measure noise floor and pressure sensitivity
- [ ] Test temperature drift compensation
- [ ] Compare bare inlet and wind-rosette performance
- [ ] Demonstrate force-feedback self-calibration prototype
- [ ] Deploy GPS-synchronized multi-node sensor array

---

## Data Policy

Small representative waveform samples, calibration files, plots, and screenshots are stored in this repository.

Large raw waveform files should not be committed directly to Git.

```text
Small CSV / JSON / screenshots → GitHub repository
Large raw waveform files       → GitHub Releases / Git LFS / cloud storage
Large videos                   → YouTube or external demo storage
Gerber ZIP releases            → GitHub Releases
```

---

## Safety and Limitations

- This is a research and prototype instrument, not a certified emergency-warning device.
- Pressure sensor performance must be verified through calibration.
- A 24-bit ADC does not guarantee 24-bit effective resolution.
- Wind-noise reduction must be tested with the complete rosette and sensor configuration.
- Force-feedback architecture is a future research module unless experimentally implemented and validated.
- Do not expose the pressure sensor directly to rain, dust, insects, or strong pressure pulses.
- Protect all pneumatic lines from moisture blockage and leakage.
- Maintain raw data along with processed data for scientific validation.

---

## Future Scope

- GPS-synchronized multi-node infrasound array for direction finding and source location
- AI-assisted classification of explosions, thunder, volcanoes, meteors and wind noise
- Solar-powered IoT stations with LoRaWAN, 4G, NB-IoT or satellite communication
- Force-feedback active null-control pressure transducer
- Automatic in-field self-calibration and predictive sensor-health monitoring
- Fusion with seismic, weather, lightning, gas and satellite observations
- Multi-hazard early-warning and environmental-monitoring platform

---

## Team Kestrel

| Role | Team Member | Responsibility |
|---|---|---|
| Team Lead | Add Name | Project coordination and SIH presentation |
| Hardware Lead | Add Name | Sensor interface, analog front end, PCB and power design |
| Embedded Lead | Add Name | STM32 firmware, ADC acquisition, GPS, SD logging |
| Software Lead | Add Name | Dashboard, visualization and data analysis |
| Mechanical Lead | Add Name | Backing chamber, rosette, enclosure and calibration chamber |
| Research/Test Lead | Add Name | Calibration, noise testing, stability analysis and documentation |

---

## References

1. New Mexico Tech Infrasound Laboratory, low-cost differential MEMS microbarometer research.
2. INFRA-EAR: low-cost mobile platform for infrasound and geophysical monitoring.
3. CTBTO International Monitoring System: infrasound monitoring and wind-noise-reduction systems.
4. Chaparral Physics microbarometer specifications and calibration documentation.
5. ADS131M04 Texas Instruments datasheet and evaluation-module documentation.
6. STM32H743 documentation, STM32CubeIDE, CMSIS-DSP and FatFs documentation.
7. ObsPy waveform processing and MiniSEED documentation.
8. Force-feedback infrasound sensor concepts presented in modern microbarometer research.

---

## License

This project is released under the [MIT License](LICENSE).

For open-hardware files such as PCB, schematics and mechanical designs, the project may later adopt a CERN Open Hardware License.

---


> **Kestrel-InfraSense aims to transform a low-cost microbarometer into an intelligent, self-validating, field-deployable infrasound sensing platform for atmospheric research, disaster monitoring, industrial safety and long-range event detection.**
