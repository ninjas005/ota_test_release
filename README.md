# ota_test_release

Firmware binaries for an OTA update test on the **Waveshare ESP32-S3-Pico**
(ESP32-S3, 16 MB flash). Get them from
[Releases](https://github.com/ninjas005/ota_test_release/releases/latest).

Flash the board over USB once. After that it checks this repository every
minute and installs new versions by itself.

## Files in a release

| File | Purpose | Address |
|---|---|---|
| `ota_test_full.bin` | Everything in one file, for the first flash | `0x0` |
| `ota_test.bin` | Application only, used for OTA | `0x20000` |
| `bootloader.bin` | Bootloader | `0x0` |
| `partition-table.bin` | Partition table | `0x8000` |
| `ota_data_initial.bin` | OTA slot selection | `0xf000` |
| `manifest.json` | Version information read by the board | not flashed |
| `SHA256SUMS.txt` | Checksums | not flashed |

For a normal installation you only need `ota_test_full.bin`.

## Step 1: Download

From the [latest release](https://github.com/ninjas005/ota_test_release/releases/latest),
download `ota_test_full.bin` and `SHA256SUMS.txt`. Or on the command line:

```bash
curl -LO https://github.com/ninjas005/ota_test_release/releases/latest/download/ota_test_full.bin
curl -LO https://github.com/ninjas005/ota_test_release/releases/latest/download/SHA256SUMS.txt
```

Check the download (optional):

```bash
sha256sum --ignore-missing -c SHA256SUMS.txt      # Linux
shasum -a 256 ota_test_full.bin                   # macOS, compare with the file
certutil -hashfile ota_test_full.bin SHA256       # Windows, compare with the file
```

## Step 2: Connect the board

Use a USB cable that carries data, then find the port:

| System | How | Example |
|---|---|---|
| Linux | `ls /dev/ttyACM* /dev/ttyUSB*` | `/dev/ttyACM0` |
| macOS | `ls /dev/cu.*` | `/dev/cu.usbmodem1101` |
| Windows | Device Manager > Ports (COM & LPT) | `COM5` |

The version is printed on the chip's **USB Serial/JTAG** port (USB ID
`303a:1001`). If two ports appear, use that one.

On Linux, if you get "permission denied", run
`sudo usermod -aG dialout $USER` and log in again.

## Step 3: Flash

### Option A: in the browser (Chrome or Edge)

1. Open <https://espressif.github.io/esptool-js/>.
2. Under **Program**, click **Connect** and choose the port.
3. Set **Flash Address** to `0x0` and choose `ota_test_full.bin`.
4. Click **Program** and wait until it finishes.
5. Press **RESET** on the board.

### Option B: command line

Requires Python 3. This installs esptool 5:

```bash
python3 -m pip install "esptool>=5"
```

Replace the port with yours:

```bash
python3 -m esptool --chip esp32s3 --port /dev/ttyACM0 --baud 460800 \
  write-flash 0x0 ota_test_full.bin
```

On Windows use `py -m pip` and `py -m esptool`, the port `COM5`, and write the
command on one line.

To flash the separate files instead:

```bash
python3 -m esptool --chip esp32s3 --port /dev/ttyACM0 --baud 460800 write-flash \
  0x0 bootloader.bin 0x8000 partition-table.bin \
  0xf000 ota_data_initial.bin 0x20000 ota_test.bin
```

### If the board is not detected

Put it in download mode: hold **BOOT**, press and release **RESET**, release
**BOOT**. Flash again, then press **RESET**.

To start from a clean chip:

```bash
python3 -m esptool --chip esp32s3 --port /dev/ttyACM0 erase-flash
```

## Step 4: Check the version

Open the port in a serial monitor:

```bash
python3 -m pip install pyserial
python3 -m serial.tools.miniterm /dev/ttyACM0 115200
```

Exit with `Ctrl+]`. The version is printed every 5 seconds:

```text
Firmware version: 1.0.0 (partition ota_0, built ...)
```

## Automatic updates

When a newer release appears here, the board downloads it, restarts and prints
the new version. You do not need to do anything. The Wi-Fi network is fixed in
the firmware, so updates work only within its range.

If the new firmware cannot reach this repository within 5 minutes, the board
returns to the previous version and skips the faulty one.

## Troubleshooting

| Problem | Fix |
|---|---|
| No port | Try another cable or USB port, or download mode |
| `Failed to connect` | Use download mode, close other serial programs |
| Restarts repeatedly after flashing | Erase the flash, then write `ota_test_full.bin` at `0x0` |
| No output | Use the `303a:1001` port and press **RESET** |
| `Wi-Fi disconnected` repeats | The configured network is not in range |
| `Manifest HTTP status 404` | No release is available |
