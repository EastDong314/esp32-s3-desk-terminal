# Roadmap

## Phase 1: Hardware Bring-Up

- Confirm ESP-IDF or alternate firmware framework.
- Build and flash a minimal hello-world firmware.
- Bring up LCD display.
- Bring up touch input.
- Connect to Wi-Fi.
- Read and write TF card files.

## Phase 2: Market Watch

- Create a simple watchlist data model.
- Fetch Binance USDT-M perpetual futures prices over REST.
- Display symbol, price, and update time.
- Add basic error and reconnect states.
- Add mark price and funding data through WebSocket.

## Phase 3: Phrase Review

- Define phrase-card JSON format.
- Load cards from TF card.
- Build review UI.
- Save review state.

## Phase 4: Audio Recording

- Capture microphone input through ES7210.
- Write WAV files to TF card.
- Show recording duration and level meter.
- Add recording list and playback.

## Phase 5: AI Translation

- Build a backend proxy for AI translation.
- Add ESP32 translation client.
- Save translated phrases as review cards.
- Add translation history.
