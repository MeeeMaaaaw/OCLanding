One-Controller ESP8266 coin flash tool
=====================================

This package flashes the prebuilt One-Controller coin firmware and writes its
runtime configuration to the ESP8266 LittleFS partition. The Arduino sketch and
source headers are not included.

Requirements
------------
- Windows PC
- NodeMCU 1.0 / ESP-12E (4 MB flash)
- USB driver for the board
- Coin acceptor with a pulse output

Wiring
------
- Coin acceptor signal -> D6 (GPIO12) through an optocoupler.
- D7 (GPIO13) is a 3.3 V logic output:
    LOW  = no insert-coin request
    HIGH = the customer has an active insert-coin request
  Use a transistor/relay/opto if the acceptor or machine needs 12 V or a
  different polarity. Fit a pull-down resistor on D7 so the relay/input stays
  low while the ESP is booting or unpowered.
- Common ground between the ESP and the optocoupler/relay supply.

Configure
---------
Option A: copy coin_config.ini.example to coin_config.ini, edit it, then run
flash.bat. The tool reads coin_config.ini automatically.

Option B: run flash.bat and answer the prompts.

Device credentials are created in the admin UI under Coin -> Create device. The
device key is shown only once. The tool does not create devices automatically.

Configuration fields
--------------------
wifi_ssid          WiFi network name
wifi_pass          WiFi password (blank for an open network)
controller_base    Controller base URL, e.g. http://172.17.0.2:4000
device_id          dev_... from the admin UI
device_key         Device HMAC key from the admin UI
vendo_id           Vendo number
machine_id         Machine id, usually m1
coin_pin           Coin pulse input (default D6)
set_pin            Insert-coin request output (default D7)
state_poll_ms      How often to poll for a request (default 2000)

Flash
-----
Double-click flash.bat, or run:

  .\flash.ps1 -Port COM5
  .\flash.ps1 -ConfigFile .\coin_config.ini -Port COM5

Options
-------
  -DryRun        Build the config image only; do not flash.
  -ConfigOnly    Update the LittleFS configuration without reflashing firmware.
  -Port COMx     Select the serial port; otherwise one is detected/prompted.
  -Baud 460800   Set the esptool upload speed.
  -NoPreserve    Skip reading the old filesystem; faster on a factory-fresh board.

The tool reads the existing LittleFS region first and preserves /coinlog.txt, so
pending coins survive a config update or reflash. Use -NoPreserve only when the
board is new or has no unposted coins.

The tool writes the firmware at 0x0 and the LittleFS image at 0x200000, matching
the NodeMCU 1.0 4 MB (FS:2 MB) layout.

After flashing
--------------
Open a serial monitor at 115200 baud. The firmware prints WiFi status, the D7
state, and one heartbeat every 10 seconds. The serial coin simulator is disabled
in this prebuilt package.

Troubleshooting
---------------
- No COM ports: install the USB driver and try another cable/port.
- Upload fails: hold FLASH while resetting, or retry at 115200 baud.
- D7 stays low: check that the controller has an active insert-coin request and
  that the ESP can reach controller_base.
- 401 from /api/coin/state: device_id, device_key, vendo_id, or machine_id does
  not match the controller.
- Old SSID or device ID persists: this package writes /coin_config.ini, but an
  Arduino IDE upload only writes the firmware. Use this package's full flash, or
  rebuild the source with OC_USE_RUNTIME_CONFIG 0 to use coin_config.h.

Bundled components
------------------
The packaged zip includes Python 3.7, pyserial, esptool, and mklittlefs from the
esp8266 Arduino core and esp-quick-toolchain. See THIRD_PARTY_NOTICES.txt and
vendor/licenses for the bundled licenses.
