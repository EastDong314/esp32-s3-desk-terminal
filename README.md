# ESP32-S3 Desk Terminal

A desktop terminal project for the LCKFB SZPI ESP32-S3 board.

The device is intended to combine four small applications:

- Binance USDT-M perpetual futures price watch
- English word and phrase review
- AI translation
- Local audio recording

## Target Hardware

- LCKFB SZPI ESP32-S3 development board
- ESP32-S3-WROOM-1-N16R8
- 16 MB Flash
- 8 MB PSRAM
- 2.0 inch 320x240 ST7789 LCD
- FT6336 capacitive touch
- ES7210 audio ADC with onboard microphones
- ES8311 audio DAC and speaker output
- TF card slot
- Wi-Fi

## Planned Applications

### Market Watch

Display Binance USDT-M perpetual futures data, starting with public mark price data.

Initial symbols:

- BTCUSDT
- ETHUSDT
- SOLUSDT

Suggested data source:

- REST: `https://fapi.binance.com/fapi/v1/ticker/price?symbol=BTCUSDT`
- WebSocket: `wss://fstream.binance.com/ws/btcusdt@markPrice@1s`

### Phrase Review

Review English words and trading-related phrases from local TF card data.

Planned actions:

- Next card
- Known
- Unknown
- Favorite
- Review due cards

### AI Translation

Translate short phrases or sentences through a small backend proxy service.

The ESP32 firmware should not store third-party AI API keys directly. Use a backend proxy for OpenAI or other AI providers.

### Audio Recording

Record microphone input through ES7210 and save WAV files to the TF card.

Planned output path:

```text
/recordings/YYYY-MM-DD_HHMMSS.wav
```

## Proposed Architecture

```text
ESP32-S3 firmware
  LCD + touch UI
  Wi-Fi client
  Binance market data client
  phrase card storage
  recording manager
  translation client

Optional backend proxy
  AI translation endpoint
  API key management
  optional Binance aggregation/cache
```

## Development Notes

The firmware stack is not fixed yet. ESP-IDF is the preferred candidate because the board documentation and audio/display examples are likely to map cleanly to ESP-IDF components.

## Status

Initialized project documentation only. Firmware and backend implementation are pending.
