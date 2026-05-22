### 3. ESP32 Firmware Responsibility
The ESP32 should act as a **transparent or intelligent gateway**. 
* It boots up, initializes **Bluetooth Low Energy (BLE)** or Classic Bluetooth, and waits for the mobile app to connect.
* When it receives a CRASH packet from the STM32, it uses the mobile app as a gateway to send an SMS/Call via the phone's cellular network, or utilizes an MQTT/HTTP request if the ESP32 is connected to a Wi-Fi hotspot.

---

## ⚙️ Software Flow & Crash Logic

To make this a true "BlackBox," your STM32 firmware should implement a **Circular Buffer** in its RAM or directly on the SD Card.



1. **Normal Mode:** 
   * The IMU is sampled at 100 Hz ($100$ times per second).
   * Data is continuously written to a 15-second circular buffer in RAM. Older data is constantly overwritten.
   * GPS and CAN bus data are sampled at 1 Hz and appended to the buffer log.
2. **Crash Detection (The Trigger):**
   * The STM32 continuously calculates the vector magnitude of acceleration:
     $$A_{mag} = \sqrt{A_x^2 + A_y^2 + A_z^2}$$
   * If $A_{mag}$ exceeds a calibrated threshold (typically between $3g$ to $5g$ for a vehicular crash), the system enters **Emergency Mode**.
3. **Emergency Mode (Snapshot Lock):**
   * The STM32 stops overwriting the circular buffer. It saves the **10 seconds of data *before* the crash** and continues recording for **5 seconds *after* the crash**.
   * This 15-second "Snapshot" is flushed permanently to a dedicated file on the MicroSD card (e.g., CRASH_001.CSV) for post-accident forensics.
   * A high-priority interrupt signals the ESP32 to send the emergency alert.
   * The buzzer sounds, and the secondary LED flashes rapidly.

---

## 🛠️ Step-by-Step Implementation Strategy

Don't try to wire everything at once. Build it modularly:

* **Phase 1: Brain & Senses:** Wire the STM32 to the MPU6500 via SPI/I2C. Write code to read raw data, convert it to G-forces, and print it to your PC terminal. Verify that slapping the sensor triggers your crash threshold.
* **Phase 2: The Storage:** Connect the MicroSD card module via SPI. Implement the circular buffer logic. Ensure you can write text files flawlessly.
* **Phase 3: The Wireless Bridge:** Connect the STM32 UART to the ESP32 UART. Write a basic script for the ESP32 to echo whatever it receives from the STM32 to its own serial monitor. Once working, implement your JSON packet parsing.
* **Phase 4: GPS & CAN:** Add the NEO-6M GPS (requires UART) and the MCP2515 CAN module. Integrate their data streams into your main telemetry packet. 
* **Phase 5: Mobile Integration:** Develop a basic mobile app (using Flutter, React Native, or MIT App Inventor) that connects to the ESP32 over Bluetooth to receive and parse the crash data.

Would you like to focus on the specific configuration of the STM32 UART DMA registers/HAL code for handling the communication, or should we look at the specific wiring layout for these components first?
