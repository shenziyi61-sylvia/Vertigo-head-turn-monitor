# Vertigo

Vertigo is an EGR 101 prototype for measuring upright, side-to-side head turns. The goal is to quantify how far, how fast, and how often the head turns from a user-defined neutral position. The live monitor displays angular velocity, relative angle, peak angle for each turn, and turn frequency. A leftward or rightward sweep counts as one turn.

## Components

| Component | Role |
| --- | --- |
| Seeed XIAO nRF52840 Sense and onboard LSM6DS3 IMU | A rigid head mount lets the three-axis gyroscope and three-axis accelerometer measure head motion. |
| Arduino firmware | Reads the IMU and sends timestamped samples to a subscribed Bluetooth Low Energy (BLE) client. |
| Python desktop monitor | Reads six-axis IMU samples over USB serial, estimates head-turn motion, and plots the results live. It can also save samples and calculated angles to CSV. |

## Source files

### [VertigoBLE.ino](VertigoBLE.ino)

This Arduino sketch runs on the Seeed XIAO BLE Sense. It advertises as `VertigoSense` and sends samples at approximately 50 Hz once a BLE client subscribes. Each 16-byte, little-endian packet contains a 32-bit microsecond timestamp followed by six signed 16-bit readings in the order `gx`, `gy`, `gz`, `ax`, `ay`, `az`. Gyroscope readings are scaled by 100 (degrees per second) and accelerometer readings by 1,000 (g).

To upload it in the Arduino IDE, select **Seeed XIAO BLE Sense** from the **Seeed nRF52 mbed-enabled Boards** package and install the **ArduinoBLE** and **Seeed Arduino LSM6DS3** libraries.

### [vertigo_live.py](vertigo_live.py)

The desktop monitor uses a stationary, upright calibration to estimate gyro bias and identify the vertical axis from gravity. It projects the gyro readings onto that axis and integrates the corrected rate to estimate the turn angle relative to neutral. Face forward after calibration and click **Zero angle** to set that reference. The angle can drift over time, and tilting or nodding can reduce accuracy.

Install the dependencies and try the monitor with simulated data:

```sh
python3 -m pip install matplotlib pyserial
python3 vertigo_live.py --demo
```

For live measurements, connect a board running firmware that emits text lines over USB serial at 115200 baud. Each line needs `gx`, `gy`, `gz`, `ax`, `ay`, and `az` fields, with an optional `t_us` timestamp. The monitor expects gyro readings in degrees per second by default. Replace the example port with your board's port:

```sh
python3 vertigo_live.py --port /dev/cu.usbmodem101 --csv session.csv
```

**Connection note.** The Arduino sketch in this repository sends binary BLE notifications. The Python monitor reads text over USB serial. Running the two together requires a BLE-to-serial bridge that decodes the packets into the expected text format. Alternatively, use firmware that streams those fields directly over USB serial.
