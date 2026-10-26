# Connected Worker Safety Alert

ESP32-based prototype that combines RFID identification, motion sensing, loud-sound detection, an OLED status display, a buzzer, and a manual help button.

## Repository layout

```text
Connected-Worker-Safety-Alert/
├── firmware/ConnectedWorkerSafetyAlert/ConnectedWorkerSafetyAlert.ino
├── tests/I2C_Scanner/I2C_Scanner.ino
├── tests/RFID_Test/RFID_Test.ino
├── tests/MPU6050_OLED_Test/MPU6050_OLED_Test.ino
├── docs/Connected-Worker-Safety-Alert-Wiring.png
├── docs/TROUBLESHOOTING.md
└── README.md
```

## Hardware

- ESP32 DevKit (classic ESP32)
- 0.96-inch I2C OLED (SSD1306, normally address `0x3C`)
- MPU6050 accelerometer/gyroscope
- RC522 RFID reader and a card/tag
- INMP441 I2S microphone
- Active buzzer compatible with the ESP32 GPIO output (use a transistor driver if the buzzer draws too much current)
- Momentary push button
- Breadboard and jumper wires

## Pin connections

| Component pin | ESP32 pin |
|---|---|
| OLED VCC / GND | 3V3 / GND |
| OLED SDA / SCL | GPIO 21 / GPIO 22 |
| MPU6050 VCC / GND | 3V3 / GND |
| MPU6050 SDA / SCL | GPIO 21 / GPIO 22 |
| RC522 3.3V / GND | 3V3 / GND |
| RC522 SDA/SS | GPIO 5 |
| RC522 SCK / MISO / MOSI | GPIO 18 / GPIO 19 / GPIO 23 |
| RC522 RST / IRQ | GPIO 27 / not connected |
| INMP441 VDD / GND | 3V3 / GND |
| INMP441 SCK / WS / SD | GPIO 26 / GPIO 25 / GPIO 33 |
| INMP441 L/R | GND (left channel) |
| Active buzzer + / - | GPIO 4 / GND, only if electrically compatible |
| Push button terminals | GPIO 32 / GND |

All modules must share GND. RC522 and INMP441 are 3.3V modules: do not power them from 5V. Do not use ESP32 GPIO 6–11; they are typically connected to flash memory. The 220-ohm resistor is not needed because this build has no LEDs.

## Software requirements

1. Install Arduino IDE.
2. Install Espressif's **esp32** board package in Boards Manager.
3. Select your ESP32 DevKit board and its serial port.
4. Install these libraries from Library Manager:
   - MFRC522 by GithubCommunity / Miguel Balboa
   - Adafruit MPU6050
   - Adafruit Unified Sensor
   - Adafruit SSD1306
   - Adafruit GFX Library
5. Open `firmware/ConnectedWorkerSafetyAlert/ConnectedWorkerSafetyAlert.ino` and upload it.
6. Open Serial Monitor at **115200 baud**.

The firmware uses the ESP32 legacy I2S driver (`driver/i2s.h`). If your installed ESP32 board package reports I2S API errors, use an Arduino-ESP32 release that provides the legacy driver or port the microphone portion to the `ESP_I2S.h` API for your installed version.

## Configure your RFID card

1. Upload the sketch and open Serial Monitor at 115200 baud.
2. Scan your RFID card. The monitor prints its UID, for example `Card UID: A1 B2 C3 D4`.
3. In the main sketch, replace the example line:

   ```cpp
   const byte AUTHORIZED_UID[] = {0xDE, 0xAD, 0xBE, 0xEF};
   ```

   with the bytes from your own card, then upload again. Keep the number of bytes the same as the card UID length.

## Behaviour

- A matching RFID UID displays **Access granted** and gives a short beep.
- An unrecognized UID displays **Access denied** and gives a longer beep.
- A strong acceleration change can trigger an **Abnormal motion** alert.
- A loud sound above the configured microphone threshold can trigger a **Loud sound detected** alert.
- Pressing the button triggers a **Manual help button** alert.
- Sensor readings and card UIDs are printed to Serial Monitor for setup and calibration.

## Calibrate before use

`MOTION_DELTA_MPS2` and `SOUND_THRESHOLD` are starting values, not universal settings. Watch the Serial Monitor in your actual environment and tune them to avoid missed events and false alarms. The loud-sound detector is a simple level threshold, not speech recognition or a certified acoustic alarm detector.

## Safety limitation

This is an educational prototype and must not be the only safety mechanism for real workers. It has no network connectivity, remote notification, battery monitoring, or certified emergency response. Test every sensor independently before combining the system.
