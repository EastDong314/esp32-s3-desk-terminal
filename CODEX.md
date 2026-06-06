# Codex Notes

## Project Goal

Build an ESP32-S3 desktop utility terminal for:

- Binance USDT-M perpetual futures price display
- English word and phrase review
- AI translation through a backend proxy
- Audio recording to TF card

## Working Principles

- Prefer board-specific examples from the LCKFB SZPI ESP32-S3 documentation before introducing generic drivers.
- Keep API keys out of firmware. Use a backend proxy for AI translation.
- Start with simple REST polling for market data, then upgrade to WebSocket when the UI and Wi-Fi flow are stable.
- Store user data on TF card where practical.
- Keep firmware modules small and testable.

## Likely Milestones

1. Board bring-up: LCD, touch, Wi-Fi, TF card.
2. LVGL home screen with app navigation.
3. Binance price watch with REST polling.
4. Local phrase review data model and UI.
5. WAV recording through ES7210 to TF card.
6. Backend proxy for AI translation.
7. Translation UI and phrase-card save flow.
8. WebSocket market data upgrade.

## Hardware References

- Board documentation: https://wiki.lckfb.com/zh-hans/szpi-esp32s3/
- Product page: https://lckfb.com/project/detail/lckfb-esp32-s3-va?param=baseInfo

## Open Decisions

- Firmware framework: ESP-IDF, Arduino, or PlatformIO.
- UI toolkit: raw drawing or LVGL.
- Backend runtime: Node.js, Python, or Cloudflare Worker.
- Whether market data should be fetched directly from the device or via backend cache.
