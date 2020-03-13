# Running WASM on ESP32

Testing WebAssembly on IoT devices.

Running WASM on ESP32 using Arduino programming style, WASM interpreter is [wasm3](https://github.com/wasm3/wasm3).

Forked from [alvarowolfx/wasm-arduino-wifi](https://github.com/alvarowolfx/wasm-arduino-wifi).

## File structure
- `rust`, `tinygo`, `assemblyscript`: Application and glue code in different high-level languages.
- `src`: Main logic to create interpreter instance and execute Wasm.
- `web`, `include`, `lib`, `test`: Other sources.
- `data`: SPIFFS temporary folder, which contains the to-be-OTAed WASM file.
- `platformio.ini`: Configuration file of Platform IO, which contains the used library.
## Usage

Tested with ESP32-DevKitC, whose MCU is `ESP32-WROOM-32`.

### Compile & Burn
1. Install PlatformIO IDE.
2. OTA update uses Wi-Fi connection. SSID and PSW should be updated to yours in `rust/app.rs` or `tinygo/app.go`.
3. Execute `build.sh` in `rust` or `tinygo` directory to build initial [SPIFFS](https://docs.platformio.org/en/latest/platforms/espressif32.html#uploading-files-to-file-system-spiffs) file system image.
4. Compile & burn using PIO built-in tools.
5. Upload initial SPIFFS image to board using `platformio run --target uploadfs`.

### OTA Update WASM file

The file is being saved on SPIFFS and the Arduino is loading from it to run on WASM3 VM. So we can update the WASM code without changing the host code.

#### Prepare *.wasm which is compiled from different languages
##### For Rust and TinyGo
* `cd` into `rust`/`tinygo` folder and run `build.sh`.
  * This will compile the Rust/TinyGo code to Wasm and also copy the file to the `data` folder.

##### For AssemblyScript (untested)

Preliminary: install `npx` and `assemblyscript` compiler.

`sudo npm i -g npx`

`npm install --save-dev AssemblyScript/assemblyscript`

Then:
* `cd` into `assemblyscript` folder and run `npm run build`.
  * This will compile the AS code to Wasm and also copy the file to the `data` folder.

#### Upload/OTA methods
##### To update locally via cable
* Run the command `pio run --target uploadfs`.

##### To update via WiFi
* Your device needs to sucessfully connect to the network. Then follow the steps:
* First search for the MDNS service using `dns-sd -B _ota .` on a PC which is under the same WiFi with ESP32. Example output:
```
Browsing for _ota._tcp
DATE: ---Thu 23 Jan 2020---
 0:04:57.636  ...STARTING...
Timestamp     A/R    Flags  if Domain               Service Type         Instance Name
 0:07:10.852  Add        2   8 local.               _ota._tcp.           esp-38A4AE30
```
* Then send an Wasm file on the /upload endpoint on the device:
```
curl -X POST -F "app.wasm=@./app.wasm" http://esp-38A4AE30.local/upload
```
* Reboot ESP32 to see serial logs.
