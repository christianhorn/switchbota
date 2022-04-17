# SwitchbOTA
Replaces the default firmware on the SwitchBot Plug Mini via OTA, enabling the use of Tasmota without disassembling the unit.

Similar to [Espressif2Arduino](https://github.com/khcnz/Espressif2Arduino).

## Disclaimer
Writing to the bootloader over OTA is dangerous and not normally done for good reason. If your device loses power or somehow flashes corrupted data, the device will be bricked and require disassembly to reprogram the device. Only perform this process if you are comfortable disassembling your plug to fix it if it breaks! Of course, do not unplug or otherwise disturb the plug while it is performing the OTA.

This is a proof of concept and provided with no warranty.

## Pre-reqs
In order to have the factory firmware download our custom firmware, you'll need to modify your local DNS to serve the desired binary. Specifically, `www.wohand.com` should point to a local machine running the included [web server](/server).

## Install
1. Setup the [web server](/server) to serve the desired OTA binaries to your device.
2. Build the [ESP-IDF binary](/espressif) or download the Release binary to flash the device.
3. Trigger an update via the product's app.

## Process
The included ESP-IDF code is a lightweight OTA client that directly writes to the embedded flash chip, enabling the install of non-app level code, including modfiying the bootloader and partiton table.

1. The update is triggered by the app with a BLE message.
2. The device fetches `http://www.wohand.com/version/wocaotech/firmware/WoPlugUS/WoPlugUS_VXX.bin` for the firmware.
3. The request is intercepted and served by our web server, which downloads the espressif binary
4. The factory firmware installs the binary to ota_0 or ota_1, depending on past usage.
5. The binary runs from its OTA partition. If it's on ota_1, it skips to step 6. Otherwise, it performs another OTA, downloading itself into ota_1 and rebooting.
6. Once on OTA_1, the device fetches `http://www.wohand.com/payload.bin` for OTA.
7. The request is intercepted and served by our web server, delivering our desired payload.
8. The espressif binary wipes the internal flash and flashes the payload. It performs two checksums, one of the downloaded binary and another of the flashed binary. It will continue to retry without restarting until the checksums are valid.

## Troubleshooting
WiFi configuration is read from the device's NVS memory by the Espressif binary. This should be configured in the SwitchBot app prior to the process. If, for whatever reason, it is lost, the binary will make a fallback connection to SSID `switchbota`, password `switchbota`. You may need to create this SSID to recover the device.
