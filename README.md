# Edge-Computing Gesture Lab Platform (Arduino Nano 33 BLE Sense + Edge Impulse)

This repository contains a **wearable edge-computing gesture recognition system and educational lab platform** built around the **Arduino Nano 33 BLE Sense Rev2** and **Edge Impulse**.

The system:

- Captures real-time inertial data for four gestures: **Pinch, Pan, Lift, Idle**
- Uses **FFT-based features** and a **compact neural network** trained in Edge Impulse
- Runs inference entirely **on-device** with **< 20 ms** latency
- Integrates into a **modular lab bench** so students can learn the full edge AI pipeline:
  data collection → signal processing → model training → deployment → evaluation

It is designed primarily as a **teaching tool for undergraduate edge computing / TinyML**, but can also serve as a reference design for gesture-controlled interfaces.

---

## 1. Project Overview

Modern edge AI usually feels like a “black box” to students: data goes in, predictions come out, and most of the magic happens in the cloud.

This project tackles that by building a **hands-on wearable platform** where students can:

- Record their own motion data using a wrist-worn device  
- Upload and process data in **Edge Impulse Studio**  
- Train a TinyML model for gesture recognition  
- Deploy the model **back onto the same device**  
- See predictions live on the display and observe latency and accuracy in real time  

The result is an **end-to-end, reproducible edge AI workflow** that runs fully on a microcontroller.

---

## 2. Features

- **Wearable IMU gesture device**
  - Based on **Arduino Nano 33 BLE Sense Rev2** with built-in 9-axis IMU (BMI270 + BMM150)
  - Battery-powered with on-board charging and regulation
  - **OLED display** for menu navigation and classification feedback
  - **MicroSD card** logging for data collection

- **Four predefined gestures**
  - **Pinch** – fingers closing towards the thumb  
  - **Pan** – lateral wrist motion  
  - **Lift** – upward wrist movement  
  - **Idle** – still hand, no motion

- **Edge Impulse–based ML pipeline**
  - Sampling at ~**90–93 Hz**
  - Segmented into **1 second non-overlapping windows** (4 windows per 4 s recording)
  - **FFT (spectral) features** extracted on all six IMU axes  
  - Compact neural network classifier trained and deployed via **Edge Impulse**

- **On-device performance**
  - Gesture classification accuracy **> 98%**
  - Test-set accuracy reported up to **≈ 99.7%** on 624 unseen windows  
  - On-device inference latency **< 20 ms** per 1 s window

- **Educational lab bench**
  - Docking board / platform that turns the wearable into a **classroom lab station**
  - Designed for repeated experiments: new data collection, retraining, and experiments with different impulse designs and models

---

## 3. Hardware Overview

Core components (as used in the project):

- **Arduino Nano 33 BLE Sense Rev2**
  - ARM Cortex-M4F @ 64 MHz
  - 1 MB Flash, 256 KB RAM
  - Onboard BMI270 accelerometer and BMM150 magnetometer
  - Native **Edge Impulse / TinyML** support

- **Sensors**
  - Built-in **9-axis IMU** (accelerometer + gyroscope + magnetometer)
  - IMU sampling configured for a **target 100 Hz**, measured effective rates around **91–93 Hz**

- **Peripherals**
  - **0.96" SSD1306 OLED (I²C)** – menus + gesture predictions
  - **MicroSD card module (SPI)** – raw CSV logging of IMU data
  - **Tactile push button** – menu navigation and recording control
  - **Li-Po battery (3.7 V)** + charging/regulation circuit – portable, long runtime

- **Enclosure and lab bench**
  - Custom PCB integrating MCU, power, SD, and display
  - 3D-printed wearable enclosure
  - Lab bench / dock to hold the device and expose connectors for classroom experiments


---

## 4. Dataset & Data Col
