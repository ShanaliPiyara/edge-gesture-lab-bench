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

## 4. Dataset & Data Collection

The dataset consists of **IMU recordings of four gestures** performed by users, recorded using the wearable.

### 4.1 Recording protocol

1. User wears the device on the wrist.
2. OLED menu offers a **“Record Gesture Data”** option.
3. Once selected, the device:
   - Samples IMU at target **100 Hz** (effective ~90–93 Hz)
   - Logs 6 channels (AccelX/Y/Z, GyroX/Y/Z) to a **CSV** on the microSD card
   - Records for **4 seconds per trial**, then automatically stops
4. Gestures are performed following a **reference video** to standardize motion:
   - Pinch, Pan, Lift, Idle

Each recording yields:

- ~**400 samples per 4 s** gesture file  
- Later segmented into **1 s windows → 4 windows per gesture trial**

### 4.2 Preprocessing

For each gesture file:

- **Structural checks**
  - Ensure 400 rows, continuous timestamps, all 6 IMU channels present  
- **Time-domain inspection**
  - Signals plotted (MATLAB / Python) to check for flatlines, spikes, and noise
- **Statistical profile**
  - Per-axis mean, standard deviation, min, max, and range
  - Used to detect sensor clipping, idle segments, drift, etc.

### 4.3 Segmentation

- **Window length:** 1000 ms (1 s)  
- **Stride:** 1000 ms (non-overlapping)  
- **Windows per 4 s file:** 4  
- Each window becomes one **training sample** of shape `(N_samples, 6)` where `N_samples ≈ 90–93`.

---

## 5. Machine Learning Pipeline (Edge Impulse)

All model development is done in **Edge Impulse Studio**.

### 5.1 Signal processing

- **Input:** 6-axis IMU time-series windows (`AccelX/Y/Z`, `GyroX/Y/Z`)  
- **Sampling frequency:** explicitly set to **≈ 90.9 Hz** in Edge Impulse to match measured data  
- **Processing block:** **Spectral Analysis (FFT)**  
  - FFT computed for each axis  
  - Only the useful portion of the spectrum retained (leveraging symmetry for real signals)  
  - **Normalization disabled** → raw magnitudes preserved to keep motion amplitude meaningful  
  - **No decimation** → full time resolution remains

### 5.2 Model

- **Type:** Small fully connected neural network (multiclass classifier)
- **Input:** Concatenated FFT feature vector
- **Output:** Softmax over 4 gesture classes:
  - `IDLE`, `LIFT`, `PAN`, `PINCH`
- **Training:**
  - Loss: categorical cross-entropy
  - Optimizer: Adam
  - Trained and tuned directly in Edge Impulse Studio

### 5.3 Performance

(Replace with the exact numbers from your EI dashboard if needed.)

- Validation accuracy: **≈ 99%**
- Test-set accuracy on **624 windows:** **≈ 99.7%**  
- Weighted precision, recall, and F1-score: **≈ 1.00**
- Only a **small number of misclassifications** (e.g. a Lift window misread as Pinch)
- **On-device inference:**  
  - Latency **< 20 ms** per 1 s window on Arduino Nano 33 BLE Sense Rev2  

---

## 6. On-Device Application Logic

The wearable firmware follows this main loop:

1. **Initialization**
   - Set up I²C, SPI, serial, OLED, IMU, SD card
   - Display a boot message and menu on the OLED

2. **Menu**
   - Options such as:
     - Record gesture data
     - Run live classification
     - (Any additional modes you’ve implemented)

3. **Data acquisition mode**
   - For recording:
     - Log IMU data to CSV on SD for 4 seconds
     - Return to menu when done

4. **Inference mode**
   - Continuously read IMU samples into a 1 s buffer
   - Wrap buffer into an `EI_SIGNAL_T` (Edge Impulse signal)
   - Call `run_classifier(&signal, &result, false);`
   - Read output probabilities and pick the top class
   - Display gesture label on OLED (and/or LEDs / serial output)

The same core model is used both for **offline dataset generation** and **live real-time gesture recognition**.

