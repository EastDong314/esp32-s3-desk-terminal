# Codex Notes

## Project Goal

Build a geek ESP32-S3 desktop terminal for:

- Binance USDT-M perpetual futures price display
- English word and phrase review
- AI translation through a backend proxy
- Audio recording to TF card

The product should be local-first and feel like a miniature always-on terminal. It should not become a phone app copied onto a 320x240 screen.

## Working Principles

- Prefer board-specific examples from the LCKFB SZPI ESP32-S3 documentation before introducing generic drivers.
- Keep API keys out of firmware. Use a backend proxy for AI translation.
- Start with simple REST polling for market data, then upgrade to WebSocket when the UI and Wi-Fi flow are stable.
- Store user data on TF card where practical.
- Keep firmware modules small and testable.
- Treat Binance as public market display only. No trading UI, no private Binance keys, no order APIs.
- Make state credibility visible: live, stale, offline, backend error, TF missing, recording failed.
- Prefer local usability over cloud dependency. Phrase review and recording should work offline.

## Likely Milestones

1. Board bring-up: LCD, touch, Wi-Fi, TF card.
2. LVGL home screen with app navigation.
3. Binance price watch with REST polling.
4. Local phrase review data model and UI.
5. WAV recording through ES7210 to TF card.
6. Backend proxy for AI translation.
7. Translation UI and phrase-card save flow.
8. WebSocket market data upgrade.
9. IMU gestures, speaker alerts, LAN debug page, and terminal themes.

## Design Direction

- 320x240 landscape terminal UI.
- Fixed status bar plus action bar.
- Dark high-contrast palette.
- Monospace price, time, and counter text.
- Dense but readable dashboard.
- Three navigation levels maximum: `Home -> App -> Detail / Action`.

## Local Data Layout

```text
/config/settings.json
/phrases/cards.json
/phrases/review_state.json
/translations/history.jsonl
/recordings/YYYY-MM-DD_HHMMSS.wav
/recordings/index.json
```

Use JSONL for append-heavy history. For state files, write through a temporary file and rename.

## Backend Direction

Early:

```text
ESP32 -> Binance REST
ESP32 -> Backend /translate
ESP32 -> TF card local storage
```

Middle:

```text
ESP32 -> Backend REST API
Backend -> Binance WebSocket aggregation
Backend -> AI Provider
Backend -> DB / Object Storage
```

Recommended backend stack when needed: Node.js + Fastify + Redis + PostgreSQL + S3/R2-compatible storage.

## Hardware References

- Board documentation: https://wiki.lckfb.com/zh-hans/szpi-esp32s3/
- Product page: https://lckfb.com/project/detail/lckfb-esp32-s3-va?param=baseInfo

## Open Decisions

- Firmware framework: ESP-IDF, Arduino, or PlatformIO.
- UI toolkit: raw drawing or LVGL.
- Backend runtime: Node.js, Python, or Cloudflare Worker.
- Whether market data should be fetched directly from the device or via backend cache.
- Pairing/token strategy for backend access.
- How much text input is realistic on the device before adding voice or LAN input.

## Key Reference Docs

- Product spec: `docs/PRODUCT_SPEC.md`
- Roadmap: `docs/ROADMAP.md`
