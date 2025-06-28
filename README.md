# ESP32 RFID Attendance System with 20×4 I²C LCD & Firebase

> **Track attendance in real time—no PC required!**
> An ESP32 reads RFID tags via an MFRC‑522 reader, shows feedback on a 20×4 I²C LCD, and logs every scan to Firebase Realtime Database (or Firestore) over Wi‑Fi.

![Project banner](PHOTO-1.jpg)
![Project banner](PHOTO-2.jpg)
![Project banner](PHOTO-3.jpg)



---

## Table of Contents

1. [Features](#features)
2. [Hardware](#hardware)
3. [Circuit Diagram](#circuit-diagram)
4. [Software Prerequisites](#software-prerequisites)
5. [Firebase Setup](#firebase-setup)
6. [Build & Flash](#build--flash)
7. [How It Works](#how-it-works)
8. [Customization](#customization)
9. [Troubleshooting](#troubleshooting)
10. [Roadmap](#roadmap)
11. [License](#license)

---

## Features

* **Touch‑less attendance** using RFID cards/fobs (13.56 MHz ISO/IEC 14443‑A).
* **Live feedback** on a 20×4 LCD: user name, status, date & time.
* **Automatic time‑sync** via NTP (no RTC crystal required).
* **Instant cloud logging** to Firebase (Realtime DB or Firestore) with JSON payloads.
* **Config file** (`config.h`) keeps Wi‑Fi & Firebase secrets out of source control.
* Built with **Arduino framework**—compatible with PlatformIO and Arduino IDE.

---

## Hardware

| Qty        | Component                          | Notes                         |
| ---------- | ---------------------------------- | ----------------------------- |
| 1          | **ESP32 DevKitC / WROOM / WROVER** | 30‑pin or 38‑pin boards work  |
| 1          | **RC522 RFID Reader**              | SPI interface, 3.3 V logic    |
| 1          | **20×4 LCD** + I²C backpack        | Based on PCF8574 A/D          |
| ≥1         | RFID tags/cards                    | MIFARE Classic 1K recommended |
| 1          | Breadboard + jumper wires          |                               |
| 1          | Micro‑USB cable                    | For power & flashing          |
| *Optional* | Piezo buzzer, RGB LED              | Audible/visual feedback       |

> ⚠️ **Logic levels:** The RC522 is 3.3 V tolerant—never feed it 5 V signals.

---

## Circuit Diagram

Fritzing PNG and `.fzz` are in `docs/circuit/`.  Below is the pin mapping used in `src/main.ino`:

| Function | ESP32 Pin | RC522 Pin  | LCD Pin (I²C bus) |
| -------- | --------- | ---------- | ----------------- |
| SPI SCK  | GPIO 18   | **SCK**    | –                 |
| SPI MISO | GPIO 19   | **MISO**   | –                 |
| SPI MOSI | GPIO 23   | **MOSI**   | –                 |
| SPI SS   | GPIO 21   | **SDA/SS** | –                 |
| RST      | GPIO 22   | **RST**    | –                 |
| I²C SDA  | GPIO 4   | –          | **SDA**           |
| I²C SCL  | GPIO 5   | –          | **SCL**           |
| Buzzer   | GPIO 27   | –          | –                 |

Feel free to adjust pins in `config.h` if your board layout differs.

---

## Software Prerequisites

### Arduino IDE / PlatformIO

* **ESP32 board package** ≥ 2.0.15
  *Arduino IDE:* `Tools → Board → ESP32 Arduino → ESP32 Dev Module`
* **Libraries** (install via Library Manager or `platformio.ini`):

  * `MFRC522` by GithubCommunity (@miguelbalboa)
  * `LiquidCrystal_I2C` by Marco Schwartz (or **LCD\_I2C** fork)
  * `WiFi` (bundled with ESP32 cores)
  * `NTPClient` by Taranais (time sync)
  * `Firebase ESP32` by Mobizt (or `Firebase-ESP-Client`)

### Clone the Repo

```bash
git clone https://github.com/your‑user/attendance-system-using-ESP32.git
cd attendance-system-using-ESP32
```

---

## Firebase Setup

1. **Create a Firebase project** at [https://console.firebase.google.com/](https://console.firebase.google.com/).
2. In **Build → Realtime Database** (or **Firestore**) click **Create Database**.
3. For Realtime DB set rules (dev):

   ```json
   {
     "rules": {
       ".read": true,
       ".write": true
     }
   }
   ```

   > Restrict rules in production!
4. **Register a web app** to obtain **API Key** and **Database URL**.
5. Generate a **Database Secret** (Realtime DB) or **Service Account key** (Firestore) and copy it.
6. Duplicate `include/config.example.h` → `include/config.h` and paste:

   ```cpp
   #define WIFI_SSID      "MyWiFi"
   #define WIFI_PASSWORD  "SuperSecret123"
   #define FB_HOST        "your‑project‑id.firebaseio.com"
   #define FB_AUTH        "xxxxxxxxxxxxxxxxxxxxxxxx"
   #define TIMEZONE_GMT   5.5   // Asia/Kolkata
   ```

---

## Build & Flash

### Arduino IDE

1. Select the correct **COM port** and **board**.
2. `Sketch → Upload`.



## How It Works

1. **Boot‑up**: ESP32 connects to Wi‑Fi, syncs NTP, mounts internal FS.
2. **Idle loop**: LCD shows date & time (`DD‑MM‑YYYY HH:MM`).
3. **Card detected**:

   * UID is read; code checks `cards.json` or cloud for user profile.
   * LCD prints *WELCOME, Ajay R24EMOO4!* and attendance status (`IN`/`OUT`) for 2 s.
   * JSON payload is pushed to `/attendance/{yyyy}/{mm}/{dd}`:

     ```json
    {
  "attendance": {
    "-OTomr21HhJ_FJCgivMq": {
      "first_mark": true,
      "late": true,
      "name": "Ajay R24EM004",
      "time": "09:58",
      "uid": "B1F4473C"
    },
    "-OTomteaQ2M_wTHFr1yL": {
      "first_mark": false,
      "late": true,
      "name": "Ajay R24EM004",
      "time": "09:58",
      "uid": "B1F4473C"
    }
  }
}
     ```
4. **Buzzer/LED** gives audible/visual acknowledgement.

> **Fail‑safe:** If internet is down, events buffer in SPIFFS and sync when online.

---

## Troubleshooting

| Symptom                     | Possible Cause        | Fix                                               |
| --------------------------- | --------------------- | ------------------------------------------------- |
| LCD shows blocks            | Wrong I²C address     | Run scanner & edit `config.h`                     |
| `RFID no card present`      | Loose SDA/SS wire     | Check wiring, level‑shift if 5 V board            |
| Firebase `401 Unauthorized` | Wrong `FB_AUTH` token | Regenerate Database Secret                        |
| Reboots after scan          | Brown‑out             | Power ESP32 from stable 5 V USB or buck converter |

More issues? [Open an issue](../../issues) with logs.

---

## Roadmap

* [ ] OTA updates via Firebase Hosting
* [ ] Encrypted storage of offline logs
* [ ] Web dashboard with charts (Vue.js + Chart.js)

---

## License

Released under the **MIT License**—see [LICENSE](LICENSE) for details.

---

> *Built with passion by [https://github.com/ajayLegion](https://github.com/ajayLegion) — Pull requests are welcome!*
