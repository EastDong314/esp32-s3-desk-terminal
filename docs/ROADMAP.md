# Roadmap

## Phase 1: Local Board Bring-Up

- Confirm ESP-IDF version, preferably `5.1.4` for parity with official examples unless there is a documented reason to use another version.
- Build and flash a minimal hello-world firmware.
- Confirm Flash and PSRAM initialization.
- Initialize shared I2C on GPIO1/GPIO2.
- Initialize PCA9557 before LCD, speaker amplifier, or camera use.
- Bring up ST7789 LCD and GPIO42 PWM backlight.
- Bring up FT6336 touch input.
- Connect to Wi-Fi.
- Read and write TF card files through SDMMC 1-bit mode on GPIO48/GPIO47/GPIO21.
- Validate ES7210 audio capture on I2S0.
- Validate ES8311/ES8310-family audio playback and PA_EN amplifier control.
- Validate QMI8658 basic motion reads at I2C address `0x6A`.

## Phase 2: App Shell

- Build fixed status bar and bottom action bar.
- Build home dashboard with four app entries.
- Add global states for Wi-Fi, TF card, backend, market freshness, and recording.
- Keep navigation within `Home -> App -> Detail / Action`.

## Phase 3: Market Watch MVP

- Keep feature display-only with no trading controls.
- Create a simple watchlist data model.
- Fetch Binance USDT-M perpetual futures prices over REST.
- Display symbol, price, and update time.
- Add stale, offline, loading, and error states.
- Start with BTCUSDT, ETHUSDT, and SOLUSDT.

## Phase 4: Phrase Review MVP

- Define phrase-card JSON format.
- Load cards from TF card.
- Build review UI.
- Save review state.
- Support Known, Again, and Favorite.

## Phase 5: Audio Recording MVP

- Capture microphone input through ES7210.
- Write WAV files to TF card.
- Show recording duration and level meter.
- Add recording list.
- Add clear TF missing, storage full, mic error, saving, and saved states.

## Phase 6: AI Translation MVP

- Build a backend proxy for AI translation.
- Add ESP32 translation client.
- Return structured JSON, not Markdown.
- Save translated phrases as review cards.
- Add translation history.
- Do not store AI provider keys in firmware.

## Phase 7: Backend Sync

- Add phrase-card sync.
- Add recording metadata upload.
- Add per-device token auth.
- Add translation cache and rate limits.

## Phase 8: Market Upgrade

- Add mark price, index price, funding rate, and next funding time.
- Add backend Binance WebSocket aggregation or ESP32 WebSocket after stability testing.
- Add stale-data age and source metadata.

## Phase 9: Geek Features

- Add QMI8658 gestures: shake refresh, flip dim or mute, tilt card switch.
- Add speaker alerts for market movement thresholds.
- Add LAN debug page for settings, watchlist, phrase upload, and recording download.
- Add terminal skins and night mode.
