# Firmware

## ZSPARK-InfraSense STM32 Firmware

This folder contains firmware for the STM32H743-based acquisition and control system.

The firmware is responsible for:

```text
- ADS131M04 ADC acquisition
- SPI + DMA transfers
- GPS PPS timestamping
- microSD waveform logging
- Temperature and environmental sensor reading
- ADC sample conversion
- Pressure calibration
- Temperature compensation
- Digital signal processing
- Event detection
- USB live waveform streaming
- System health monitoring
- Optional force-feedback control
```

---

# 1. Firmware Architecture

```text
ADS131M04 ADC
      │
      ▼
SPI + DRDY Interrupt + DMA
      │
      ▼
STM32H743 Acquisition Task
      │
      ▼
Circular Sample Buffer
      │
      ├── SD Logging Task
      ├── DSP Task
      ├── Event Detection Task
      ├── USB Communication Task
      ├── GPS Timing Task
      ├── Environmental Sensor Task
      └── System Health Task
```

---

# 2. Development Environment

| Tool | Purpose |
|---|---|
| STM32CubeIDE | Firmware development and debugging |
| STM32CubeMX | Pin, clock and peripheral configuration |
| STM32 HAL | Peripheral drivers |
| FreeRTOS | Task scheduling |
| CMSIS-DSP | FFT, filters, RMS and signal processing |
| FatFs | microSD file system |
| ST-LINK | Flashing and debugging |
| Git | Version control |

---

# 3. Recommended Hardware

```text
Development board: NUCLEO-H743ZI2
ADC: ADS131M04EVM or ADS131M04 custom board
GPS: u-blox module with PPS
RTC: DS3231
Temperature: TMP117
Humidity: SHT45
Absolute pressure: BMP390
Accelerometer: ADXL355
Storage: microSD card
```

---

# 4. Firmware Folder Structure

```text
firmware/
│
├── README.md
│
├── stm32h743/
│   ├── ZSPARK_InfraSense.ioc
│   ├── Core/
│   │   ├── Inc/
│   │   └── Src/
│   │
│   ├── Drivers/
│   ├── Middlewares/
│   │
│   ├── App/
│   │   ├── adc_acquisition/
│   │   ├── calibration/
│   │   ├── communications/
│   │   ├── dsp/
│   │   ├── environmental_sensors/
│   │   ├── event_detection/
│   │   ├── force_feedback/
│   │   ├── gps_timing/
│   │   ├── sd_logging/
│   │   └── system_health/
│   │
│   ├── Config/
│   └── tests/
│
├── libraries/
│   ├── ads131m04_driver/
│   ├── tmp117_driver/
│   ├── sht45_driver/
│   ├── bmp390_driver/
│   ├── adxl355_driver/
│   ├── gps_driver/
│   └── ds3231_driver/
│
└── tools/
    ├── flash_firmware.sh
    ├── serial_debug.py
    └── decode_packet.py
```

---

# 5. Peripheral Configuration

| Peripheral | Purpose |
|---|---|
| SPI | ADS131M04 communication |
| DMA | Continuous ADC frame transfer |
| GPIO interrupt | ADC DRDY signal |
| GPIO output | ADC reset and chip select |
| UART | GPS NMEA data |
| GPIO interrupt | GPS PPS timing signal |
| I2C | TMP117, SHT45, BMP390, DS3231, INA226 |
| SPI/I2C | ADXL355 accelerometer |
| SDMMC | microSD data logging |
| USB CDC | Laptop real-time data output |
| Timer | Sample timing and diagnostics |
| Watchdog | System recovery |
| PWM/DAC | Future actuator/force-feedback path |

---

# 6. FreeRTOS Task Structure

| Task | Priority | Function |
|---|---:|---|
| ADC Acquisition Task | Highest | Reads ADC frames after DRDY event |
| Buffer Task | High | Stores ADC samples in circular buffer |
| SD Logging Task | High | Writes waveform blocks to microSD |
| GPS PPS Task | High | Maintains timestamp synchronization |
| DSP Task | Medium | Filters, FFT, RMS and PSD |
| Event Task | Medium | STA/LTA and event metadata |
| USB Task | Medium | Sends live packets to dashboard |
| Environmental Task | Low | Reads sensor status and weather data |
| Health Task | Low | Monitors ADC, SD, GPS, voltage and faults |
| Display Task | Low | Updates OLED screen |
| Calibration Task | Low | Handles test/calibration mode |
| Force Feedback Task | Future | PID actuator control |

---

# 7. ADC Acquisition Flow

```text
ADS131M04 completes conversion
        │
        ▼
DRDY interrupt occurs
        │
        ▼
STM32 starts SPI DMA transfer
        │
        ▼
ADC frame arrives in DMA buffer
        │
        ▼
Convert 24-bit values to signed 32-bit values
        │
        ▼
Store sample frame in circular buffer
        │
        ▼
Notify SD logging and DSP tasks
```

---

# 8. 24-bit ADC Sample Conversion

The ADC sends a 24-bit signed number.

```c
int32_t adc_code;

adc_code = ((int32_t)rx << 16) |
           ((int32_t)rx[1] << 8)  |
           ((int32_t)rx);[2]

if (adc_code & 0x800000) {
    adc_code |= 0xFF000000;
}
```

The final pressure conversion is based on calibration coefficients.

```text
Pressure = (ADC_Value - Offset) / Sensitivity
```

Temperature compensation is applied after initial conversion.

```text
Pressure_Corrected =
[Pressure_Measured - Offset(Temperature)] / Sensitivity(Temperature)
```

---

# 9. Data Packet Format

The STM32 sends structured packets to the laptop dashboard.

```text
+--------------------------------------------------------+
| Header                                                 |
+--------------------------------------------------------+
| Sample Counter                                         |
+--------------------------------------------------------+
| GPS / RTC Timestamp                                    |
+--------------------------------------------------------+
| Pressure Channel 0                                     |
+--------------------------------------------------------+
| Pressure Channel 1                                     |
+--------------------------------------------------------+
| Vibration / Channel 2                                  |
+--------------------------------------------------------+
| Calibration / Diagnostic Channel 3                     |
+--------------------------------------------------------+
| Temperature Sensor 1                                   |
+--------------------------------------------------------+
| Temperature Sensor 2                                   |
+--------------------------------------------------------+
| Humidity                                               |
+--------------------------------------------------------+
| Wind Speed                                             |
+--------------------------------------------------------+
| Battery Voltage                                        |
+--------------------------------------------------------+
| Status Flags                                           |
+--------------------------------------------------------+
| CRC                                                    |
+--------------------------------------------------------+
```

---

# 10. Data Logging

## Raw waveform storage

The firmware should store raw data before filtering.

```text
Raw ADC data
+ Timestamp
+ Sensor configuration
+ Gain setting
+ Calibration version
+ Environmental metadata
```

## Logging rule

```text
Do not write one sample at a time to microSD.
```

Instead:

```text
1. Store samples in RAM buffer.
2. Collect a block of samples.
3. Write block to microSD using SDMMC.
4. Keep a second buffer active during write.
```

This prevents microSD write delays from interrupting ADC acquisition.

---

# 11. DSP Features

| Feature | Purpose |
|---|---|
| Band-pass filter | Analyze 0.01–20 Hz infrasound |
| Low-pass filter | Reject high-frequency noise |
| RMS calculation | Measure pressure/noise level |
| FFT | Identify dominant frequency |
| PSD | Measure noise floor |
| Spectral features | Support event classification |
| STA/LTA trigger | Detect sudden event energy |
| Cross-correlation | Compare pressure/reference channels |
| Wind correlation | Identify wind-driven noise |
| Vibration correlation | Identify mechanical artifacts |

---

# 12. Build and Flash Procedure

## Required tools

```text
STM32CubeIDE
ST-LINK driver
NUCLEO-H743ZI2
USB cable
ADS131M04EVM or custom ADC board
```

## Build steps

```text
1. Clone repository.
2. Open firmware/stm32h743/ZSPARK_InfraSense.ioc in STM32CubeIDE.
3. Confirm target board/MCU.
4. Generate CubeMX code if configuration changed.
5. Build project.
6. Connect ST-LINK USB.
7. Flash firmware.
8. Open serial terminal.
9. Verify boot messages.
10. Verify ADC and sensor initialization.
```

---

# 13. First Firmware Milestones

```text
Milestone 1:
LED blink and UART serial output

Milestone 2:
ADS131M04 SPI register read

Milestone 3:
DRDY interrupt and continuous ADC samples

Milestone 4:
ADC data output over USB/UART

Milestone 5:
TMP117 and DS3231 I2C read

Milestone 6:
GPS NMEA and PPS detection

Milestone 7:
microSD write test

Milestone 8:
Continuous waveform logging

Milestone 9:
Live USB waveform packets

Milestone 10:
RMS, FFT and STA/LTA event trigger
```

---

# 14. Safety and Reliability Rules

```text
- Enable independent watchdog.
- Use ADC communication timeout detection.
- Detect microSD write failure.
- Store error flags in health log.
- Do not block inside interrupt handlers.
- Do not process FFT inside ADC interrupt.
- Use DMA and FreeRTOS queues/tasks.
- Keep raw data separate from processed data.
- Use calibration version number in every recorded file.
```

---

# 15. Force-Feedback Firmware Note

The `force_feedback/` module is for future work.

It may include:

```text
- PID controller
- DAC command generation
- Voice-coil current driver control
- Current monitor
- Position sensor input
- Self-calibration current injection
- Null-position error monitoring
```

This module should remain disabled until the passive differential sensor system is calibrated and working.
