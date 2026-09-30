# Software

## ApeX-InfraSense Dashboard and Analysis Software

This folder contains the laptop-side software for:

- Real-time waveform visualization
- Serial packet reception
- Pressure data decoding
- FFT and spectrum display
- Spectrogram generation
- Event monitoring
- Calibration analysis
- Noise-floor analysis
- Frequency-response plotting
- CSV/MiniSEED conversion
- Report generation

---

# 1. Software Architecture

```text
STM32 USB / Ethernet Output
           │
           ▼
Python Packet Receiver
           │
           ▼
Packet Decoder
           │
           ▼
Live Data Buffer
           │
           ├── Waveform Plot
           ├── FFT / Spectrum Plot
           ├── Spectrogram
           ├── RMS / Peak Display
           ├── Environmental Panel
           ├── GPS / SD Health Panel
           └── Event Monitor
```

---

# 2. Folder Structure

```text
software/
│
├── README.md
│
├── dashboard/
│   ├── app.py
│   ├── config.py
│   ├── serial_receiver.py
│   ├── packet_decoder.py
│   ├── data_buffer.py
│   ├── waveform_plot.py
│   ├── spectrum_plot.py
│   ├── spectrogram.py
│   ├── event_monitor.py
│   ├── health_monitor.py
│   ├── export_data.py
│   └── requirements.txt
│
├── analysis/
│   ├── calibration_analysis.py
│   ├── frequency_response.py
│   ├── phase_response.py
│   ├── sensitivity_analysis.py
│   ├── noise_psd.py
│   ├── temperature_compensation.py
│   ├── stability_analysis.py
│   ├── wind_noise_analysis.py
│   └── miniseed_export.py
│
├── simulation/
│   ├── pneumatic_filter_model.py
│   ├── capillary_response_model.py
│   ├── force_feedback_pid_model.py
│   └── wind_rosette_model.py
│
└── notebooks/
    ├── 01_sensor_calibration.ipynb
    ├── 02_frequency_response.ipynb
    ├── 03_noise_floor.ipynb
    ├── 04_temperature_drift.ipynb
    └── 05_field_test_analysis.ipynb
```

---

# 3. Required Python Packages

Create `software/dashboard/requirements.txt` with:

```text
numpy
scipy
matplotlib
pyserial
pyqtgraph
pandas
plotly
obspy
```

Optional packages:

```text
fastapi
uvicorn
influxdb-client
dash
pytest
jupyter
```

---

# 4. Installation

## Create Python environment

```bash
python -m venv .venv
```

## Activate environment

### Windows

```bash
.venv\Scripts\activate
```

### Linux/macOS

```bash
source .venv/bin/activate
```

## Install packages

```bash
pip install -r software/dashboard/requirements.txt
```

---

# 5. Live Dashboard Features

The dashboard should show:

```text
- Live pressure waveform
- Raw waveform
- Filtered waveform
- Current pressure value
- RMS pressure/noise
- Peak pressure
- FFT spectrum
- Spectrogram
- Dominant frequency
- Sensor temperature
- Backing chamber temperature
- Humidity
- Wind speed
- Vibration level
- GPS lock status
- UTC time
- microSD status
- Battery voltage
- Event trigger status
- Calibration version
```

---

# 6. Dashboard Startup

Example command:

```bash
python software/dashboard/app.py
```

Example serial configuration:

```text
Serial port: COM3 or /dev/ttyACM0
Baud rate: 921600
Packet format: Binary packet with CRC
```

---

# 7. Data Processing Pipeline

```text
Raw STM32 packet
        │
        ▼
CRC validation
        │
        ▼
Packet decoding
        │
        ▼
ADC counts to pressure conversion
        │
        ▼
Temperature correction
        │
        ▼
Band-pass filter: 0.01–20 Hz
        │
        ├── RMS calculation
        ├── FFT
        ├── PSD
        ├── Spectrogram
        ├── STA/LTA event trigger
        ├── Wind correlation
        └── Vibration correlation
        │
        ▼
Live display + file storage
```

---

# 8. Analysis Scripts

| Script | Purpose |
|---|---|
| `calibration_analysis.py` | Converts calibration recordings into coefficients |
| `frequency_response.py` | Generates frequency-response graph |
| `phase_response.py` | Generates phase-response graph |
| `sensitivity_analysis.py` | Calculates counts/Pa or V/Pa |
| `noise_psd.py` | Calculates noise RMS and PSD |
| `temperature_compensation.py` | Fits offset/sensitivity thermal model |
| `stability_analysis.py` | Calculates drift over time |
| `wind_noise_analysis.py` | Compares bare inlet vs rosette |
| `miniseed_export.py` | Exports waveform to MiniSEED |

---

# 9. Waveform Data Format

Recommended raw packet fields:

```text
timestamp_utc
sample_counter
pressure_ch0_raw
pressure_ch1_raw
vibration_ch2_raw
diagnostic_ch3_raw
temperature_sensor_board
temperature_backing_chamber
humidity
wind_speed
battery_voltage
gps_status
sd_status
event_status
calibration_version
```

Recommended processed CSV fields:

```text
timestamp_utc
pressure_raw_pa
pressure_corrected_pa
pressure_filtered_pa
pressure_rms_pa
dominant_frequency_hz
temperature_c
humidity_percent
wind_speed_mps
vibration_rms
battery_voltage
event_flag
event_confidence
```

---

# 10. Real-Time Filtering

For live display, use a causal filter.

```text
Suggested analysis band:
0.01 Hz to 20 Hz
```

For offline analysis, zero-phase filtering may be used.

```text
Offline filtering should never replace stored raw waveform data.
```

---

# 11. Event Detection

The first event detector uses STA/LTA.

```text
STA/LTA = Short-Term Average / Long-Term Average
```

Suggested starting configuration:

```text
STA window: 5 seconds
LTA window: 120 seconds
Trigger ratio: 3.0
Release ratio: 1.5
```

These are initial values only and must be tuned from real noise data.

---

# 12. Dashboard Development Order

```text
Step 1:
Receive serial data from STM32.

Step 2:
Decode pressure channel.

Step 3:
Plot live pressure waveform.

Step 4:
Save data to CSV.

Step 5:
Show temperature, wind and vibration.

Step 6:
Add RMS and peak pressure.

Step 7:
Add FFT spectrum.

Step 8:
Add spectrogram.

Step 9:
Add event trigger display.

Step 10:
Add calibration and sensor-health display.
```

---

# 13. Offline Analysis Workflow

```text
1. Download raw data from microSD.
2. Convert raw binary data to CSV.
3. Apply calibration coefficients.
4. Apply temperature compensation.
5. Generate waveform plot.
6. Generate FFT and PSD.
7. Generate spectrogram.
8. Calculate noise floor.
9. Generate frequency-response plot.
10. Export event data and report.
```

---

# 14. Data Storage Rule

```text
Raw data must always be preserved.

Do not overwrite raw waveform data with filtered data.
```

Use this structure:

```text
data/
├── raw/
├── processed/
├── calibration/
├── events/
└── reports/
```

---

# 15. Future Software Features

```text
- Web dashboard using FastAPI or Flask
- Grafana long-term monitoring
- InfluxDB time-series database
- Automatic report generation
- Machine-learning event classification
- Multi-node array cross-correlation
- Direction-of-arrival estimation
- Cloud event notification
- Mobile alert application
```
