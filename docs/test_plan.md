# Test Plan

## ZSPARK-InfraSense Verification and Validation Plan

This document defines the test plan for the ZSPARK-InfraSense infrasound microbarometer.

The objective is to verify that the complete system performs as an infrasound sensor, not only as an electronic circuit.

The system will be tested for:

- Sensor communication
- ADC operation
- Pneumatic leakage
- Frequency response
- Sensitivity
- Noise floor
- Temperature drift
- Long-term stability
- Wind-noise reduction
- Data logging
- GPS timing
- Real-time waveform display
- Event detection

---

# 1. Test Summary

| Test ID | Test Name | Main Purpose |
|---|---|---|
| T-01 | Power-on self-test | Verify system starts correctly |
| T-02 | ADC communication test | Verify SPI and DRDY operation |
| T-03 | Pressure sensor test | Verify sensor responds to pressure |
| T-04 | Pneumatic leakage test | Verify chamber/tubing is airtight |
| T-05 | ADC noise test | Measure digitizer noise |
| T-06 | Analog power-noise test | Verify clean analog supply |
| T-07 | Frequency-response test | Verify 0.01–20 Hz response |
| T-08 | Sensitivity test | Estimate counts/Pa or V/Pa |
| T-09 | Phase-response test | Measure timing/phase behavior |
| T-10 | Linearity test | Verify proportional measurement |
| T-11 | Noise-floor test | Measure RMS and PSD |
| T-12 | Temperature-drift test | Measure thermal offset effect |
| T-13 | 24-hour stability test | Verify short-term stability |
| T-14 | 7-day stability test | Verify long-term stability |
| T-15 | Wind-rosette test | Measure wind-noise reduction |
| T-16 | GPS timing test | Verify PPS timestamp accuracy |
| T-17 | microSD logging test | Verify continuous data storage |
| T-18 | Dashboard test | Verify real-time waveform display |
| T-19 | Event detection test | Verify STA/LTA event trigger |
| T-20 | System integration test | Verify complete end-to-end operation |

---

# 2. Test Environment

## Laboratory environment

```text
- Stable table
- No nearby fans or air-conditioner outlet
- Low electrical interference
- Shielded analog wiring
- Stable power supply
- Controlled room temperature where possible
```

## Outdoor environment

```text
- Sensor mounted away from vibrating machinery
- Wind sensor installed near rosette
- Pipe rosette laid symmetrically
- Enclosure protected from direct rain
- GPS antenna has sky visibility
```

---

# 3. T-01: Power-On Self-Test

## Objective

Verify that the system starts and detects required hardware.

## Procedure

```text
1. Apply power.
2. Observe power LED.
3. Verify STM32 starts.
4. Verify ADC response.
5. Verify microSD mount.
6. Verify RTC read.
7. Verify GPS UART data.
8. Verify environmental sensor communication.
9. Verify display status.
```

## Pass criteria

```text
- No fault LED
- ADC detected
- microSD mounted
- RTC returns valid time
- Temperature and humidity values received
- GPS data received or GPS status shown as acquiring
```

---

# 4. T-02: ADC Communication Test

## Objective

Verify ADS131M04 communication with STM32 through SPI.

## Procedure

```text
1. Power ADC board.
2. Reset ADC using STM32 GPIO.
3. Read ADC status/ID register.
4. Configure sample rate.
5. Enable continuous conversion.
6. Monitor DRDY signal.
7. Read sample frames using SPI + DMA.
8. Print sample values through UART/USB.
```

## Pass criteria

```text
- ADC status register responds
- DRDY appears continuously
- No SPI frame errors
- Sample values update at configured rate
- No missed-sample counter increase
```

---

# 5. T-03: Pressure Sensor Response Test

## Objective

Verify that the pressure sensor responds to a small controlled pressure difference.

## Procedure

```text
1. Connect sensor P1 and P2 to equal pressure.
2. Record baseline output.
3. Apply a small pressure change using syringe/piston.
4. Observe pressure output.
5. Reverse pressure direction.
6. Confirm output polarity changes correctly.
```

## Pass criteria

```text
- Output changes when pressure is applied
- Output returns near baseline after equalization
- Positive and negative pressure produce opposite output direction
- No unexpected saturation
```

---

# 6. T-04: Pneumatic Leakage Test

## Objective

Verify that chamber, capillary and pneumatic fittings are leak-tight.

## Procedure

```text
1. Seal pneumatic path.
2. Apply small pressure step using syringe.
3. Hold pressure.
4. Record pressure decay over time.
5. Repeat for each capillary configuration.
```

## Pass criteria

```text
- No sudden pressure drop
- Decay matches expected capillary equalization behavior
- No audible leaks
- No loose fittings
- No pressure loss from chamber seal
```

---

# 7. T-05: ADC Noise Test

## Objective

Measure ADC and analog front-end noise with stable or shorted input.

## Procedure

```text
1. Disconnect pressure sensor or connect stable reference.
2. Short/zero ADC input using approved test condition.
3. Record at least 30 minutes.
4. Calculate standard deviation and RMS noise.
5. Plot PSD.
```

## Pass criteria

```text
- No periodic switching spikes
- No excessive 50/60 Hz pickup
- ADC noise is lower than expected pressure sensor noise
- No saturation events
```

---

# 8. T-06: Analog Power-Noise Test

## Objective

Verify that analog supply rails are clean.

## Procedure

```text
1. Measure analog LDO output using oscilloscope.
2. Measure digital supply rail.
3. Compare analog rail with Wi-Fi/display active and inactive.
4. Observe ripple and switching noise.
```

## Pass criteria

```text
- Analog rail remains stable
- No major periodic ripple in infrasound band
- Digital loads do not create visible pressure-signal artifacts
- No ground-loop noise observed
```

---

# 9. T-07: Frequency-Response Test

## Objective

Verify sensor response from 0.01 Hz to 20 Hz.

## Procedure

```text
1. Place sensor and reference sensor in calibration chamber.
2. Apply sine-wave pressure.
3. Test each required frequency.
4. Measure amplitude ratio.
5. Measure phase difference.
6. Plot response.
```

## Test frequencies

```text
0.01, 0.02, 0.05, 0.1, 0.2, 0.5, 1, 2, 5, 10, 15, 20 Hz
```

## Pass criteria

```text
- Detectable output at all target frequencies
- No unexpected resonance
- Lower cutoff documented
- Upper response reaches 20 Hz
- Amplitude and phase results recorded
```

---

# 10. T-08: Sensitivity Test

## Objective

Determine conversion from pressure to ADC output.

## Procedure

```text
1. Apply known pressure amplitude.
2. Record reference pressure.
3. Record ADC counts.
4. Calculate counts/Pa.
5. Repeat at several amplitudes.
```

## Pass criteria

```text
- Sensitivity remains reasonably consistent
- Calibration coefficients are stored
- Pressure conversion produces realistic values
```

---

# 11. T-09: Phase-Response Test

## Objective

Measure delay introduced by sensor, pneumatic path and filters.

## Procedure

```text
1. Record reference and test-sensor sine waves.
2. Measure time difference.
3. Convert time delay to phase difference.
4. Repeat across frequency range.
```

## Pass criteria

```text
- Phase response documented
- No unexplained discontinuity
- Phase delay is repeatable
```

---

# 12. T-10: Linearity Test

## Objective

Verify proportional response over pressure range.

## Procedure

```text
1. Select a test frequency, such as 1 Hz.
2. Apply multiple pressure amplitudes.
3. Measure output.
4. Plot reference pressure versus measured pressure.
5. Calculate error.
```

## Pass criteria

```text
- Measured pressure follows reference pressure trend
- No severe saturation
- Maximum error documented
```

---

# 13. T-11: Noise-Floor Test

## Objective

Measure total system noise.

## Procedure

```text
1. Equalize P1 and P2 pressure.
2. Record at least 1 hour.
3. Repeat for 24 hours.
4. Calculate RMS pressure noise.
5. Calculate PSD in multiple frequency bands.
```

## Pass criteria

```text
- RMS noise documented in Pa or mPa
- PSD graph generated
- Noise sources identified where possible
- Raw data stored
```

---

# 14. T-12: Temperature-Drift Test

## Objective

Measure temperature-induced offset and sensitivity changes.

## Procedure

```text
1. Stabilize sensor at multiple temperatures.
2. Record zero offset.
3. Apply fixed calibration signal.
4. Record sensitivity.
5. Fit correction curve.
```

## Pass criteria

```text
- Offset versus temperature graph generated
- Sensitivity versus temperature graph generated
- Compensation coefficients stored
```

---

# 15. T-13: 24-Hour Stability Test

## Objective

Verify short-term stability.

## Procedure

```text
1. Run system continuously for 24 hours.
2. Record pressure, temperature and power status.
3. Verify microSD data continuity.
4. Calculate offset drift and RMS noise.
```

## Pass criteria

```text
- No unexpected reboot
- No data loss
- No ADC communication failure
- Drift trend documented
```

---

# 16. T-14: 7-Day Stability Test

## Objective

Verify long-term stability.

## Procedure

```text
1. Deploy sensor in stable location.
2. Record continuously for seven days.
3. Monitor power, GPS, temperature and SD status.
4. Calculate daily drift and noise metrics.
```

## Pass criteria

```text
- Continuous data record available
- System health logs available
- Daily drift documented
- Environmental influence documented
```

---

# 17. T-15: Wind-Rosette Test

## Objective

Measure wind-noise-reduction performance.

## Procedure

```text
1. Record with bare sensor inlet.
2. Record with basic windscreen.
3. Record with single pipe.
4. Record with four-arm pipe rosette.
5. Record wind speed during every test.
6. Compare PSD and RMS values.
```

## Pass criteria

```text
- Rosette data shows lower wind-correlated noise than bare inlet
- Wind speed and noise relation documented
- Pipe blockage/moisture issues identified
```

---

# 18. T-16: GPS Timing Test

## Objective

Verify timestamp synchronization.

## Procedure

```text
1. Connect GPS PPS signal to STM32 interrupt pin.
2. Log PPS timestamps.
3. Compare RTC and GPS time.
4. Measure time consistency between consecutive PPS pulses.
```

## Pass criteria

```text
- GPS lock status detected
- PPS pulses recognized
- UTC timestamps stored with waveform data
- RTC backup works when GPS is disconnected
```

---

# 19. T-17: microSD Logging Test

## Objective

Verify waveform logging reliability.

## Procedure

```text
1. Start continuous recording.
2. Record for at least 4 hours.
3. Check file size and timestamps.
4. Read stored files using laptop software.
5. Verify no missing blocks.
```

## Pass criteria

```text
- Data files created correctly
- No corrupted data blocks
- File timestamps are valid
- Storage does not interrupt ADC acquisition
```

---

# 20. T-18: Dashboard Test

## Objective

Verify real-time waveform display.

## Procedure

```text
1. Connect STM32 USB serial to laptop.
2. Start dashboard.
3. Verify incoming pressure values.
4. Check waveform scrolling.
5. Check FFT and spectrogram.
6. Check environmental status display.
```

## Pass criteria

```text
- Live waveform visible
- Data updates continuously
- Dashboard shows timestamp and system status
- No serial packet loss under normal operation
```

---

# 21. T-19: Event Detection Test

## Objective

Verify automatic event trigger.

## Procedure

```text
1. Generate controlled pressure pulse.
2. Run STA/LTA event algorithm.
3. Verify event trigger.
4. Verify pre-event and post-event buffer storage.
5. Compare with wind/vibration readings.
```

## Pass criteria

```text
- Event triggers at defined threshold
- Event waveform is saved
- Event metadata includes time, RMS and peak value
- False trigger behavior is documented
```

---

# 22. T-20: Full System Integration Test

## Objective

Verify complete sensor operation.

## Procedure

```text
1. Assemble sensor, rosette, ADC, STM32 and enclosure.
2. Start full system.
3. Verify GPS, ADC, SD, dashboard and sensors.
4. Apply known calibration signal.
5. Observe live waveform.
6. Trigger event detection.
7. Save data.
8. Export data for analysis.
```

## Pass criteria

```text
- Complete signal path works
- Pressure waveform is visible
- Data is logged
- Dashboard works
- Environmental sensors work
- Calibration data can be applied
- No major communication failures
```

---

# 23. Test Results Format

Every completed test should include:

```text
Test ID:
Test Name:
Date:
Operator:
Hardware Version:
Firmware Version:
Sensor ID:
Capillary Cartridge:
Backing Chamber Volume:
ADC Sample Rate:
Temperature:
Humidity:
Wind Speed:
Procedure:
Observed Result:
Pass/Fail:
Files Generated:
Graphs Generated:
Issues Found:
Corrective Action:
```

---

# 24. Test Status Table

| Test ID | Test Name | Status | Result |
|---|---|---|---|
| T-01 | Power-on self-test | Planned | Pending |
| T-02 | ADC communication | Planned | Pending |
| T-03 | Pressure sensor response | Planned | Pending |
| T-04 | Pneumatic leakage | Planned | Pending |
| T-05 | ADC noise | Planned | Pending |
| T-06 | Analog power noise | Planned | Pending |
| T-07 | Frequency response | Planned | Pending |
| T-08 | Sensitivity | Planned | Pending |
| T-09 | Phase response | Planned | Pending |
| T-10 | Linearity | Planned | Pending |
| T-11 | Noise floor | Planned | Pending |
| T-12 | Temperature drift | Planned | Pending |
| T-13 | 24-hour stability | Planned | Pending |
| T-14 | 7-day stability | Planned | Pending |
| T-15 | Wind rosette | Planned | Pending |
| T-16 | GPS timing | Planned | Pending |
| T-17 | microSD logging | Planned | Pending |
| T-18 | Dashboard | Planned | Pending |
| T-19 | Event detection | Planned | Pending |
| T-20 | Full system integration | Planned | Pending |
