# 🚁 ESP32-Based Flight Controller for F450 Quadcopter

[![Expertise: Embedded Systems](https://img.shields.io/badge/Expertise-Embedded%20Systems-blueviolet)](#) [![Board: ESP32](https://img.shields.io/badge/Board-ESP32-red)](https://espressif.com) [![IMU: MPU6050](https://img.shields.io/badge/IMU-MPU6050-blue)](#) [![Control: PID](https://img.shields.io/badge/Control-PID%20Stabilization-green)](#) [![Frame: F450](https://img.shields.io/badge/Frame-F450%20Quadcopter-orange)](#) [![License: MIT](https://img.shields.io/badge/License-MIT-brightgreen)](#)

A fully custom ESP32-based flight controller for an F450 quadcopter, built from scratch. This project implements MPU6050 IMU integration, complementary-filter attitude estimation, PID-based stabilization, and ESC motor control — validated through real-world flight tests covering takeoff, hover, pitch, roll, and yaw maneuvers.

---

## 🏗️ System Architecture

The controller runs as a tight feedback loop at its core:

1. **IMU (MPU6050)** – Reads raw gyroscope and accelerometer data over I2C at high frequency.
2. **Sensor Fusion (Complementary Filter)** – Blends gyro integration (fast, noisy) with accelerometer angles (slow, stable) to produce clean real-time pitch and roll estimates.
3. **PID Controller** – Computes correction signals for Pitch, Roll, and Yaw axes based on the error between desired and actual attitude.
4. **Motor Mixer** – Maps PID outputs to individual motor throttle commands.
5. **ESC PWM Output** – Drives all four ESCs using ESP32 hardware timers for precise, jitter-free PWM signals.

---

## 🛠️ Hardware Configuration & Pinout

### Flight Controller (Drone)

| Component | ESP32 Pin | Function |
| --- | --- | --- |
| MPU6050 SDA | GPIO 21 | I2C Data |
| MPU6050 SCL | GPIO 22 | I2C Clock |
| Motor 1 (Front-Right) | GPIO 13 | PWM — CW |
| Motor 2 (Front-Left) | GPIO 12 | PWM — CCW |
| Motor 3 (Rear-Left) | GPIO 14 | PWM — CW |
| Motor 4 (Rear-Right) | GPIO 27 | PWM — CCW |

> Refer to the [`/Hardware`](./Hardware) folder for the full wiring schematic used in this build.

### Motor Layout (Top View — X Configuration)

```
        FRONT
  (1) CW     (2) CCW
      \         /
       \       /
        \     /
         \   /
          \ /
           X
          / \
         /   \
        /     \
  (3) CCW   (4) CW
        REAR
```

### Bill of Materials

| Component | Specification | Qty |
| --- | --- | --- |
| Microcontroller | ESP32 DevKit (38-pin) | 1 |
| Frame | F450 Quadcopter Frame + PDB | 1 |
| IMU | MPU6050 (GY-521 module) | 1 |
| BLDC Motors | 1000 KV brushless | 4 |
| ESCs | 30A PWM ESC | 4 |
| Propellers | 10×4.5 inch CW + CCW pair | 2 pairs |
| Battery | 3S LiPo 11.1V, 2200 mAh+ | 1 |
| Voltage Regulator | 5V BEC (for ESP32) | 1 |

---

## 🚀 Setup & Flashing

### 1. Arduino IDE Configuration

Add the ESP32 board to Arduino IDE via **File → Preferences → Additional Board Manager URLs**:

```
https://raw.githubusercontent.com/espressif/arduino-esp32/gh-pages/package_esp32_index.json
```

Then install **ESP32 by Espressif Systems** from **Tools → Board → Board Manager**.

### 2. Install Libraries

Install via **Arduino Library Manager**:

- `MPU6050` by Electronic Cats (or equivalent)
- `Wire` (built-in)

### 3. ESC Calibration

Before uploading the main flight code, calibrate all four ESCs:

- Remove propellers — always test without props first
- Send maximum throttle signal at power-on
- Wait for calibration beep sequence
- Bring throttle to minimum → ESCs are calibrated
- Power cycle the system

### 4. Flash the Firmware

```
# Open in Arduino IDE
Code/flight_controller.ino

# Board:       ESP32 Dev Module
# Baud Rate:   115200
# Upload
```

### 5. First Power-On Test (No Props)

- Place drone on flat surface and power on
- Wait ~2–3 seconds for MPU6050 to initialise
- Verify all 4 ESCs arm (listen for beep sequence)
- Slowly increase throttle — confirm all motors spin and respond correctly

---

## 🎛️ PID Tuning

PID gains determine how aggressively the drone corrects attitude errors. The values below are a starting point for the F450 frame — real-world tuning is always required.

### Default Values

```cpp
// Pitch
float Kp_pitch = 1.3,  Ki_pitch = 0.04,  Kd_pitch = 18.0;

// Roll
float Kp_roll  = 1.3,  Ki_roll  = 0.04,  Kd_roll  = 18.0;

// Yaw
float Kp_yaw   = 4.0,  Ki_yaw   = 0.02,  Kd_yaw   = 0.0;
```

### Tuning Steps

- **P only first** → increase Kp until the drone oscillates, then back off ~30%
- **Add D** → increase Kd until oscillations are damped
- **Add I last** → small values only, to correct slow steady-state drift
- Tune each axis (Pitch, Roll, Yaw) independently

### Common Issues

| Symptom | Cause | Fix |
| --- | --- | --- |
| Rapid oscillations | Kp too high | Reduce Kp |
| Slow drift | Ki too low | Increase Ki |
| Overshooting | Kd too low | Increase Kd |
| Flips on takeoff | Wrong motor/prop direction | Check motor layout diagram |
| One side always drops | IMU not level | Re-run IMU calibration |

---

## 📂 Repository Structure

```
├── Code/                    # Main flight controller firmware (.ino)
├── Hardware/                # Wiring schematics and connection diagrams
├── Project Photos/          # Build and assembly photos
├── FLight Test Videos/      # Real-world flight test recordings
└── Achievements/            # Project milestones and validation results
```

---

## ⚠️ Disclaimer

Always test without propellers first.  
Fly outdoors in an open area, away from people and obstacles.  
Verify motor rotation directions before attaching props.  
Use at your own risk — no responsibility for damage or injury.
