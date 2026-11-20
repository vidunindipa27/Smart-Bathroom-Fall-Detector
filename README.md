# Smart Bathroom Fall Detector (Team OFFSET)

**Course:** EN1190 – Engineering Design Project  
**Institution:** Department of Electronic and Telecommunication Engineering, University of Moratuwa

A privacy-first, non-optical bathroom fall detection system powered by 24GHz mmWave radar and an ESP32 microcontroller. Designed to mitigate the severe health risks of "long lie" scenarios among elderly or mobility-impaired individuals, the system requires zero wearable compliance and uses no cameras or microphones. Upon verifying a fall via a custom kinematic state machine, it dispatches an instant SOS alert over Wi-Fi using the Telegram Bot API.

---

## Repository Structure

```text
├── code/               # ESP32 C++ firmware (Triple-Lock state machine & Telegram dispatch)
├── report/             # Final project report (OFFSET_Final_Report.pdf)
└── pcb_and_schematics/ # Custom PCB layout, schematics, and fabrication files
```

---

## System Overview & Key Features

* **Privacy-First Sensing:** Uses an **HLK-LD2450 24GHz mmWave radar sensor** to output spatial telemetry (X/Y coordinates and radial velocity) without capturing optical video or audio.
* **"Triple-Lock" Kinematic Algorithm:** Filters out routine bathroom movements and environmental noise (such as oscillating fans or wet-tile reflections) through three sequential validation stages:
  1. **Fall Initiation:** Monitors downward radial acceleration exceeding a calibrated velocity threshold (`Vy < -110 cm/s`) to ignore normal walking or sitting.
  2. **Velocity Gradient Analysis:** Measures the deceleration profile to reach a stationary state (`< 15.0 cm/s`). Impacts must fall strictly within a **50 ms to 450 ms** window—rejecting instantaneous hand flicks (`< 50 ms`) and controlled muscle squats (`> 450 ms`).
  3. **Temporal Verification & Self-Recovery:** Enters a **6-second verification window** after impact. Sustained low-speed movement (`> 15.0 cm/s` for 1.5s) or a rapid push-up (`> 40.0 cm/s`) cancels the alarm. Complete stillness escalates the state to `ALARM`.
* **Automated Emergency Dispatch & Heartbeat:** Connects via 802.11 b/g/n Wi-Fi to send an encrypted HTTPS notification via the **Universal Telegram Bot API** within ~5.5 seconds of impact, alongside an automated 24-hour operational heartbeat message.
* **Self-Healing UART Pipeline:** Actively flushes serial buffers (`Serial2.flush()`) at 256,000 baud after blocking HTTPS handshakes to prevent frame desynchronization and buffer overflows.

---

## Hardware Architecture

* **Microcontroller:** ESP32-WROOM-32U
* **Sensor:** HLK-LD2450 24GHz mmWave Radar Module
* **Power Management:** 5V DC Barrel Jack input stepped down via an **AMS1117-3.3V** LDO regulator with decoupling capacitors (`10 µF`, `1 µF`, `100 nF`) to handle ESP32 Wi-Fi transmission current spikes.
* **Flashing & Debugging:** Dedicated header interface for a **PL2303 USB-to-TTL** UART adapter and onboard status LEDs (`DS1`, `DS2`).
* **Enclosure:** Custom 3D-printed housing engineered to keep the radar module's field of view unobstructed by metallic components.

---

## Known Issues & Hardware Errata (Rev 1.0)

* **Programmable Indicator LED (`DS2`) Non-Functional:**  
  * **Symptom:** Onboard programmable status LED `DS2` (tied to GPIO 25 for visual Doppler/alert debugging) fails to illuminate during operation.  
  * **Root Cause:** Current draw limitation / improper current-limiting configuration on the `DS2` trace/GPIO path in the current PCB revision.  
  * **Impact:** Purely cosmetic/debugging; core radar telemetry, fall detection state machine, power regulation, and Telegram SOS alerts operate reliably without interruption.  
  * **Planned Fix (Rev 2.0):** Adjust series resistor sizing or drive `DS2` via a low-side N-channel MOSFET/transistor buffer to decouple indicator current draw from the ESP32 GPIO pin.

---

## Firmware Setup & Flashing

1. **Dependencies:** Install the Arduino IDE (or PlatformIO) with ESP32 board support and the following libraries:
   * `UniversalTelegramBot`
   * `ArduinoJson`
2. **Configuration:** Open the firmware in `/code` and enter your Wi-Fi credentials (`SSID`, `PASSWORD`), Telegram `BOT_TOKEN`, and target `CHAT_ID`.
3. **Flashing:** Connect a **PL2303 USB-to-TTL** adapter to the onboard programming headers, hold the boot/flash buttons (`SW1`/`SW2`) as required, and upload the sketch.

---

## Team OFFSET (University of Moratuwa)

* **Gunawardhane D.T.** (`240207G`) – Algorithm & Firmware
* **Pelendagamage V.I.** (`240493C`) – Hardware & PCB Design
* **Samaranayaka D.M.K.I.** (`240583D`) – Physical Testing & Data Analysis
* **Manvidu E.M.K.I.** (`240421G`) – 3D Enclosure CAD & Assembly
