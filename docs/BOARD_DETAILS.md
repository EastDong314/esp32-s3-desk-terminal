# Board Details: LCKFB SZPI ESP32-S3

This document summarizes the board details verified from the official LCKFB SZPI ESP32-S3 wiki before firmware implementation.

Primary documentation entry:

- https://wiki.lckfb.com/zh-hans/szpi-esp32s3/
- https://wiki.lckfb.com/zh-hans/szpi-esp32s3/beginner/prepare.html

## Understanding Status

The official wiki has enough detail to proceed with the desk terminal design:

- ESP-IDF is the documented development flow.
- Board-level examples exist for every required subsystem in this project: LCD, LVGL, touch-related UI, Wi-Fi, TF card, ES7210 audio input, ES8311 audio output, MP3 playback, voice recognition, camera, and IMU.
- Critical board dependencies are clear: shared I2C bus, PCA9557 IO expander, ST7789 over SPI, TF in 1-bit SD mode, and I2S audio codec wiring.

The one remaining implementation discipline is to use the downloaded official example source and schematic as the final authority for exact macro names and any pin definitions not visible in the rendered wiki text.

## Documentation Map

Official pages checked:

- Download center: `download-center.html`
- Open-source hardware: `open-source-hardware/`
- Open-source software examples: `open-source-software/`
- Preparation and tutorial index: `beginner/prepare.html`
- Board introduction: `beginner/introduction.html`
- ESP-IDF environment: `beginner/install.html`
- Development flow: `beginner/design-flow.html`
- BOOT key: `beginner/key.html`
- IMU: `beginner/attitude-sensor.html`
- TF card: `beginner/sd-card.html`
- Audio input ES7210: `beginner/audio-input-es7210.html`
- Audio output ES8311: `beginner/audio-output-es8311.html`
- LCD display: `beginner/lcd-display.html`
- Camera: `beginner/camera.html`
- LVGL: `beginner/lvgl.html`
- Wi-Fi: `beginner/wifi.html`
- MP3 player: `beginner/mp3.html`
- Voice wake and command recognition: `beginner/voice-recognition.html`
- Face detection: `beginner/face-detection.html`
- FAQ: `question.html`

Official downloads:

- Full board materials are shared from the download center.
- Example source package is shared from the open-source software page.
- Hardware source and 3D enclosure links are provided from the open-source hardware page.

## Development Environment

Official flow:

- Use ESP-IDF rather than Arduino or MicroPython for the tutorial path.
- Use VSCode with the Espressif ESP-IDF extension.
- Select target chip `esp32s3`.
- Use serial download through the board USB connection.
- The official S3 examples were made with ESP-IDF `5.1.4`.
- The wiki reports the examples were also tested with ESP-IDF `5.2.2`.

Project implication:

- Start firmware with ESP-IDF.
- Prefer IDF `5.1.4` for parity with official examples unless a later dependency forces a newer version.
- Keep `.vscode/` and `build/` out of commits unless deliberately needed.
- Record the exact IDF version used for every firmware milestone.

## Core Hardware

Main module:

- ESP32-S3-WROOM-1-N16R8.
- 16 MB Flash.
- 8 MB PSRAM.
- Xtensa 32-bit LX7 dual-core, up to 240 MHz.
- 2.4 GHz Wi-Fi.
- Bluetooth 5 LE and Bluetooth Mesh.

Important constraints:

- The module uses 8-line PSRAM.
- GPIO35, GPIO36, and GPIO37 are occupied by PSRAM and should not be used.
- GPIO46 must be low in download mode. The board holds it low with a pulldown; external hardware connected to GPIO46 must not drive it high during boot/download.

## USB, Download, and Debug

USB architecture:

- Type-C connects to a CH334F USB 2.0 hub.
- One downstream port connects to CH340K USB-to-UART for UART0 download and serial terminal.
- Another downstream port connects to ESP32-S3 USB-OTG.
- CH334F needs no driver on Windows 10 and Linux according to the wiki.
- CH340K may need the WCH CH341 serial driver.

Project implication:

- One Type-C cable should support power, flashing, serial logs, and USB device experiments.
- Keep serial logging enabled during board bring-up.

## Shared I2C Bus

The board uses one shared I2C bus for multiple devices:

- QMI8658 IMU.
- FT6336 capacitive touch.
- ES7210 audio ADC.
- ES8311 or ES8310-family audio output codec path as described by the wiki.
- PCA9557 IO expander.
- External I2C GH connector.

Known I2C details from examples:

- I2C port: `0`.
- SDA: GPIO1.
- SCL: GPIO2.
- QMI8658 7-bit address: `0x6A`.
- ES7210 address on this board: `0x41`.

Project implication:

- Initialize I2C once in a shared BSP layer.
- Do not let app modules independently install/uninstall the I2C driver.
- External I2C devices must coexist on the same bus and address space.
- I2C locking is needed if multiple tasks access touch, IMU, audio codec configuration, and IO expander.

## PCA9557 IO Expander

The PCA9557 IO expander controls board signals that are mandatory for several subsystems.

Known output bits:

```text
BIT(0): LCD_CS
BIT(1): PA_EN
BIT(2): DVP_PWDN
```

Behavior:

- `LCD_CS` controls the LCD chip-select path.
- `PA_EN` low disables the speaker amplifier; high enables speaker output.
- `DVP_PWDN` high puts the camera in standby; low enables the camera.

Project implication:

- Initialization order for display, speaker, or camera must be:

```text
I2C init -> PCA9557 init -> peripheral-specific init
```

- Audio output must call the amplifier enable path only when actually playing sound or alerts.
- Camera support, if added later, must explicitly release `DVP_PWDN`.

## LCD and Backlight

Display:

- 2.0 inch IPS LCD.
- ST7789 driver.
- SPI interface.
- Native panel resolution is 240x320.
- Official example configures it for landscape-like use through XY swap and mirroring, yielding 320x240 drawing in examples.
- RGB565 is used for bitmap drawing.

Backlight:

- GPIO42 controls LCD backlight.
- GPIO42 high closes the backlight and low opens it in the board description.
- The official example drives it with LEDC PWM and output inversion.
- Example LEDC settings use low-speed mode, 10-bit duty resolution, 5 kHz PWM, and channel 0.

Official example:

- `06-lcd`
- Initializes I2C, PCA9557, then LCD.
- Uses `esp_lcd` ST7789 APIs.
- Allocates display buffers in PSRAM for full images or rows.

Project implication:

- Use LVGL with partial refresh and avoid full-screen redraws for frequently changing prices.
- Keep backlight brightness as a first-class setting for desk/night modes.
- Initialize PCA9557 before LCD because LCD CS depends on the expander.

## Touch and LVGL

Touch controller:

- FT6336 capacitive touch.
- I2C interface on the shared I2C bus.

LVGL:

- Official example `08-lcd_lvgl`.
- It starts from the LCD/camera project and adds LVGL integration.
- The example runs LVGL demos including `lv_demo_benchmark()`.
- The benchmark can be scrolled by touch, confirming touch input integration in the official flow.

Project implication:

- LVGL is the right UI path for this project.
- Build the app shell after LCD, touch, I2C, and PCA9557 are stable.
- Keep touch controls large because the panel is only 2 inches.

## TF Card

Storage:

- TF card uses 1-bit SD mode through the SDMMC peripheral.
- Required pins from official examples:

```text
CMD:  GPIO48
CLK:  GPIO47
DAT0: GPIO21
```

Electrical notes:

- DAT3 must be pulled high so the card works in SD mode rather than SPI mode.
- The wiki notes ESD protection on unused data lines because of TF card insertion/removal risk.

Official example:

- `03-sd_card`
- Mounts a filesystem and logs SDMMC use.

Project implication:

- Use TF card for all user data: phrase cards, review state, translation history, recordings, recording index, and settings.
- Test sustained writes before enabling long recordings.
- Use safe-write patterns for JSON state files.

## Audio Input: ES7210

Audio ADC:

- ES7210 audio ADC.
- The chip supports 4 microphone channels.
- The board uses 3 channels: 2 microphones plus 1 DAC feedback path for echo cancellation.
- ES7210 I2C address on this board is `0x41`.

I2S input details from official example:

```text
I2S port:       0
MCLK:           GPIO38
BCLK:           GPIO14
WS/LRCLK:       GPIO13
DATA IN:        GPIO12
Format:         ES7210_I2S_FMT_I2S
Sample rate:    48000 Hz in example
Sample bits:    16-bit
Slot mask:      slot 0 + slot 1
```

Official example:

- `04-audio_es7210`
- Based on ESP-IDF `examples/peripherals/i2s/i2s_codec/i2s_es7210_tdm`.
- Also includes SD card pins for recording output.

Project implication:

- Recording MVP should start with 16-bit mono WAV to TF card even if the hardware path captures stereo/TDM-like data.
- Audio capture must be isolated from UI rendering with a ring buffer or dedicated task.
- Use smaller UI refresh workload while recording.

## Audio Output: ES8311 / ES8310 Path and Speaker

The board documentation names ES8311 in the hardware table and describes the DAC output path as ES8310-family in the circuit explanation. For firmware work, use the official example source and board package as the naming authority.

Output path:

- Audio DAC output feeds ES7210 MIC3 for echo cancellation feedback.
- Another DAC output path feeds NS4150B class-D amplifier and the onboard 1 W speaker.
- `PA_EN` is controlled through PCA9557.
- `PA_EN = 0` disables speaker output.
- `PA_EN = 1` enables speaker output.

Official examples:

- `05-audio_es8311`
- `11-mp3_player`

MP3 example notes:

- Uses `esp-audio-player`.
- Uses `esp-libhelix-mp3` for MP3 decode.
- Example stores MP3 files in SPIFFS.
- SPIFFS partition is 3 MB in the example and does not support Chinese filenames.
- The wiki suggests switching to SD card paths for SD-based music files.

Project implication:

- Speaker alerts should initialize audio output and enable `PA_EN` only during playback.
- For recording playback or alert sounds, prefer WAV/PCM first before adding MP3 dependency.
- If MP3 is used later, store audio files on TF card rather than SPIFFS for this product.

## IMU: QMI8658

Sensor:

- QMI8658 6D IMU.
- 3-axis accelerometer plus 3-axis gyroscope.
- Supports SPI and I2C; the board uses I2C.
- 7-bit I2C address: `0x6A`.

Official example:

- `02-attitude`
- Reads acceleration and gyro registers.
- Calculates X/Y/Z tilt angles.
- Example accelerometer setup: 4 g, 250 Hz.
- Example gyroscope setup: 512 dps, 250 Hz.

Project implication:

- IMU is appropriate for geek gestures: shake refresh, flip dim/mute, tilt card navigation.
- Gestures should be debounced and treated as optional shortcuts, not the only path to core actions.

## Camera

Camera:

- GC0308.
- 300K pixels.
- BTB connector.
- Single 2.8 V supply in the board description.
- PWDN is controlled by PCA9557 `DVP_PWDN`.
- `DVP_PWDN = 1` standby.
- `DVP_PWDN = 0` work mode.

Official example:

- `07-lcd_camera`
- Uses RGB565 pixel format.
- Uses QVGA frame size, 320x240.
- Uses 2 frame buffers in the documented adaptation.

Project implication:

- Camera is not required for the current MVP.
- If future camera features are added, budget PSRAM carefully because camera frame buffers, LVGL buffers, and network buffers compete.
- Camera and LCD share the small display budget; avoid camera features until market, phrase, translation, and recording are stable.

## Wi-Fi

Official Wi-Fi example:

- `09-wifi`
- Covers scanning and connecting.

Project implication:

- Start with STA mode.
- Add reconnect and visible network states early.
- Market and AI features must show stale/offline/backend-error states.
- Phrase review and recording should work without Wi-Fi.

## Bluetooth HID

Official example:

- `10-ble_hid`

Project implication:

- BLE HID is not part of the MVP.
- It can become a later geek feature, for example sending hotkeys or acting as a small macro pad.

## AI Examples

Voice recognition:

- Official chapter uses ESP-Skainet and `esp-sr`.
- `esp-sr` provides AFE, WakeNet, MultiNet, and Chinese speech synthesis modules.
- Official speech example adds dependency `espressif/esp-sr: '~1.6.0'`.
- Feed and detect tasks are pinned to separate cores in the example flow.

Face detection:

- Official chapter demonstrates image recognition on ESP32-S3 using the camera path.

Project implication:

- Voice wake/command is feasible but should be P2, not MVP.
- Recording is the correct first audio milestone.
- AI translation should stay backend-based; ESP-SR is for local wake/command, not general translation.

## BOOT Key and User IO

The BOOT key can be used as a user key because automatic download circuitry handles download mode.

The board exposes limited general-purpose IO:

- BOOT key input.
- GPIO10 and GPIO11 on the multifunction GH connector.

Project implication:

- Do not assume many spare GPIOs are available.
- External add-ons should prefer the dedicated I2C GH connector or the GPIO10/GPIO11 multifunction connector.

## Project-Specific Bring-Up Checklist

1. Install ESP-IDF `5.1.4` or explicitly document the chosen version.
2. Build and flash Hello World with target `esp32s3`.
3. Verify serial logs through CH340K.
4. Initialize shared I2C on GPIO1/GPIO2.
5. Initialize PCA9557 and verify LCD CS, PA_EN, DVP_PWDN control functions.
6. Bring up ST7789 LCD and PWM backlight on GPIO42.
7. Bring up LVGL and FT6336 touch.
8. Mount TF card through SDMMC 1-bit mode on GPIO48/GPIO47/GPIO21.
9. Verify ES7210 capture on I2S0 pins GPIO38/GPIO14/GPIO13/GPIO12.
10. Save a short WAV recording to TF card.
11. Verify ES8311/ES8310-family output path and `PA_EN` speaker enable.
12. Connect Wi-Fi and fetch a small HTTPS response.
13. Add Binance REST polling.
14. Add phrase JSON loading and review state.
15. Add backend translation proxy calls.

## Implementation Rules for This Repository

- Use official LCKFB examples as the BSP baseline.
- Keep one shared board-support layer for I2C, PCA9557, display, touch, storage, audio, IMU, and network.
- Do not duplicate I2C initialization across app modules.
- Do not enable speaker amplifier except when playing audio.
- Use TF card, not Flash, for user data and recordings.
- Keep camera and local speech recognition out of MVP.
- Treat exact board source files from the official download as final pin-definition authority before writing production firmware.
