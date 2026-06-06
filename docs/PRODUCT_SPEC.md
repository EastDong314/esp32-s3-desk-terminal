# Product Spec: Geek ESP32-S3 Desk Terminal

## Positioning

This project is a geek desktop information and learning terminal for the LCKFB SZPI ESP32-S3 board.

It is not a reduced phone app. It should feel like a small always-on embedded terminal: dense, readable, tactile, local-first, and hackable.

The core loop is:

```text
Watch Binance USDT-M perpetual futures
  -> encounter a trading or technical phrase
  -> translate or explain it through AI
  -> save it as a phrase card
  -> review it during idle moments
  -> record thoughts, follow-up notes, or spoken practice
```

## Product Manager View

### Users

- Crypto futures watchers who want a low-distraction desk display for Binance USDT-M perpetual prices.
- Hardware geeks who enjoy ESP32-S3, LCD, touch, TF card storage, microphones, speaker output, and exposed local data.
- English learners who want to collect trading, AI, and technical phrases from real usage.
- Light AI-tool users who need short phrase translation, explanation, examples, and phrase-card generation.
- Local recording users who want fast voice notes, trade review snippets, meeting fragments, or pronunciation practice.

### Core Scenarios

- Desk market watch: show BTCUSDT, ETHUSDT, SOLUSDT and other configured symbols with price, mark price, freshness, and connection state.
- Phrase review between work sessions: show one phrase card at a time with known, again, and favorite actions.
- AI translation to card: translate a phrase, explain it, generate an example, and save it to the phrase deck.
- Quick recording: start, stop, and save WAV files to the TF card with visible recording duration and level state.
- Offline fallback: phrase review and recording continue to work without Wi-Fi or backend access.

### MVP Scope

P0 requirements:

- ESP-IDF project foundation.
- LCD display, touch input, Wi-Fi, TF card read/write.
- LVGL or equivalent app shell with a home dashboard and four apps.
- Binance USDT-M price display using REST polling first.
- Local phrase-card JSON loading and review state saving.
- ES7210 microphone recording to WAV on the TF card.
- AI translation through a backend proxy. No AI API key in firmware.
- Clear stale, offline, TF missing, backend offline, recording failed, and save failed states.

Out of MVP:

- Trading, orders, private Binance keys, positions, or account data.
- Full K-line charts, order book depth, or complex indicators.
- Full keyboard text input.
- Local speech recognition.
- AI chat.
- Long-form audio editing, denoise, or transcription.
- Cloud sync as a hard dependency.

### Success Metrics

- Device runs for 24 hours without crash, file-handle leak, or visible memory degradation.
- Market data freshness is visible at all times.
- REST market refresh remains stable in the 3-10 second range.
- Recording starts with visible feedback within 1 second.
- Stopped recordings reliably land as readable WAV files.
- Translation results can be saved as phrase cards in 2-3 actions.
- Firmware contains no AI or exchange private keys.

## Designer View

### Design Direction

The UI should look like a miniature trading and learning terminal:

- Dark, high contrast, terminal-like.
- Dense but not cramped.
- Strong status visibility.
- Monospace numbers for prices, times, and counters.
- Minimal motion and no decorative launch page.
- Color communicates state, not decoration.

Recommended palette:

```text
Background:     #050608 / #0B0F14
Panel:          #111820
Panel Border:   #25313D
Primary Text:   #E6EDF3
Secondary Text: #8B949E
Muted Text:     #59636E
Accent Cyan:    #20D6E5
Positive:       #24C47E
Negative:       #FF4D5E
Warning:        #F5C542
Error:          #FF3B30
```

### Information Architecture

```text
Home Dashboard
  Market Watch
    Watchlist
    Symbol Detail
    Connection/Error State
  Phrase Review
    Due Card
    Known / Again / Favorite
    Review Summary
  AI Translation
    Short Text Input
    Translation Result
    Save as Phrase Card
  Audio Recording
    Record Screen
    Level Meter
    Recording List
  System Layer
    Wi-Fi
    TF Card
    Backend
    Time
    Error Toast / Modal
```

### Global Layout

Target 320x240 landscape layout:

```text
+--------------------------------+
| Status Bar 320x20              |
+--------------------------------+
| Main Content 320x196           |
+--------------------------------+
| Bottom Nav / Action Bar 320x24 |
+--------------------------------+
```

Touch targets should be at least about 40x32 px. Do not place more than four primary actions in the bottom bar.

### Home Dashboard

```text
+--------------------------------+
| ESP TERM      WiFi TF  22:41   |
+--------------------------------+
| BTC  68,245.3  +1.2%           |
| ETH   3,812.7  -0.4%           |
| SOL     162.9  +3.8%           |
+---------------+----------------+
| MARKET        | PHRASE         |
| Live / 1s     | Due: 12        |
+---------------+----------------+
| TRANSLATE     | RECORD         |
| Backend OK    | Ready          |
+---------------+----------------+
```

### Market Watch

- Show symbol, price, mark price, percentage move, update age, and live/stale state.
- Use both color and arrow symbols for positive/negative movement.
- Do not show stale prices as normal.
- MVP should avoid full K-line charts. A simple sparkline can be a later enhancement.

### Phrase Review

- One card per screen.
- Phrase is largest, meaning is secondary, example is compact.
- Primary actions: `Known`, `Again`, `Favorite`.
- Long text should be clipped, paged, or converted into a shorter card by the backend.

### AI Translation

- Explicitly show backend state: `Backend OK`, `Backend Offline`, `Timeout`, `Proxy Error`.
- Return structured translation content, not long Markdown.
- Save result as a phrase card.
- MVP may use selected text, preset entries, or history instead of full text input.

### Audio Recording

- Recording screen prioritizes duration, level meter, file path/status, and stop action.
- Recording list is a separate screen.
- Stop button must be large and unambiguous.
- Supported states: `Ready`, `Recording`, `Saving`, `Saved`, `TF Missing`, `Storage Full`, `Mic Error`, `Failed`.

## Backend Architect View

### Principle

The backend is a capability-extension layer for the ESP32-S3. The ESP32 remains local-first; the backend handles secrets, expensive AI calls, optional cache, optional sync, and complex network behavior.

### Service Boundaries

Market Data Service:

- Aggregates Binance USDT-M public market data.
- Can start as direct ESP32 REST calls, then move to backend cache/WebSocket aggregation.
- Must stay display-only. No trading API, no private Binance keys.

Translation Service:

- Holds AI provider keys in server-side secrets.
- Accepts short text and returns compact structured JSON.
- Caches repeated phrase translations by hash.
- Can generate phrase-card suggestions.

Phrase Sync Service:

- Optional backup/sync for TF-card phrase data.
- Uses card IDs, `updated_at`, and tombstones for simple conflict handling.
- ESP32 local storage remains the source of offline usability.

Recording Metadata Service:

- Stores recording metadata and optional upload state.
- Audio upload is optional and should not block local recording.
- Large uploads should use object storage or presigned URLs if implemented.

### API Shape

Use small, flat JSON responses:

```json
{
  "ok": true,
  "data": {},
  "server_time": "2026-06-06T14:00:00Z"
}
```

Error response:

```json
{
  "ok": false,
  "error": {
    "code": "RATE_LIMITED",
    "message": "Too many requests"
  },
  "server_time": "2026-06-06T14:00:00Z"
}
```

Recommended endpoints:

```http
GET  /market/usdt-m/tickers?symbols=BTCUSDT,ETHUSDT,SOLUSDT
POST /translate
GET  /phrases/sync?since=<cursor>
POST /phrases/sync
POST /recordings
POST /recordings/{id}/upload-url
```

### Security

- Do not embed OpenAI, AI provider, or private exchange keys in firmware.
- Binance support remains public market data only.
- Use per-device tokens for backend access.
- Store only token hashes server-side.
- Rate-limit `/translate`, `/phrases/sync`, market batch requests, and upload endpoints.
- Validate symbols with a whitelist or strict format.
- Limit translation text length, for example 500 characters.
- Avoid logging full tokens, provider keys, long user text, or signed upload URLs.

### Deployment Path

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

Recommended full stack:

```text
Node.js + Fastify + Redis + PostgreSQL + S3/R2-compatible storage
```

Fast lightweight option:

```text
Cloudflare Worker + D1 + R2
```

## Hardware Engineer View

### Board Capabilities

Target board:

- ESP32-S3-WROOM-1-N16R8, 16 MB Flash, 8 MB PSRAM.
- 2 inch 320x240 ST7789 LCD.
- FT6336 capacitive touch.
- ES7210 audio ADC with onboard microphones.
- ES8311 audio DAC and speaker path.
- TF card slot.
- Wi-Fi.
- QMI8658 6D IMU.

The board can support LVGL UI, Wi-Fi market polling, TF-card file storage, and I2S recording, but the project should validate the board examples before building complex application logic.

### Bring-Up Order

1. ESP-IDF hello world and flashing.
2. Flash and PSRAM verification.
3. ST7789 LCD with backlight control and orientation.
4. FT6336 touch with coordinate calibration.
5. Wi-Fi STA connect and reconnect.
6. TF card mount, read/write, large sequential write.
7. ES7210 microphone capture.
8. ES8311 playback.
9. QMI8658 gesture events.

### Driver Modules

Suggested firmware boundaries:

```text
display_driver   LCD, backlight, LVGL flush
touch_driver     FT6336 events
audio_input      ES7210 capture, level meter
audio_output     ES8311 playback, tones
storage          TF card, paths, settings, safe writes
imu_driver       QMI8658 gesture events
network          Wi-Fi, HTTP, WebSocket
app_shell        home, app switching, global status
```

### Performance Constraints

- Avoid full-screen high-frequency redraws on 320x240 SPI LCD.
- Use partial updates for changing prices and recording level meter.
- Start with three market symbols.
- Keep JSON responses small and flat.
- Do not maintain several long TLS/WebSocket connections on the ESP32 if a backend can aggregate.
- During recording, prioritize audio DMA and TF-card writes over UI refresh richness.

### Storage Plan

Use TF card as the user data disk:

```text
/config/settings.json
/phrases/cards.json
/phrases/review_state.json
/translations/history.jsonl
/recordings/YYYY-MM-DD_HHMMSS.wav
/recordings/index.json
```

WAV sizing:

```text
16 kHz / 16-bit / mono ~= 32 KB/s
1 minute ~= 1.9 MB
1 hour ~= 115 MB
```

Use JSONL for append-heavy history. For state JSON files, write to a temporary file and rename to reduce corruption risk.

## Geek View

### Geek Features

- Funding countdown on the home dashboard.
- Mark price, index price, and last price comparison.
- Stale-data badge and update age display.
- Price movement flash and speaker tone alerts.
- Shake to refresh market data.
- Flip to dim or mute.
- Tap gesture to start voice note.
- Tilt to switch phrase cards.
- Local TF-card data openness: users can edit phrase JSON and export recordings directly.
- LAN debug page for status, watchlist editing, phrase upload, and recording download.
- Terminal skins: dense terminal, pixel market screen, high-contrast night mode.

### Experimental Modes

- Trading desk mini-TV: always-on futures watch panel.
- Translation card machine: translate, explain, and convert phrases into review cards.
- Voice note block: timestamped recordings for trading review or spoken practice.
- Offline phrase trainer: learning continues without Wi-Fi.
- Audio market alarm: speaker alert when movement threshold is crossed.

## State Model

Global states:

```text
Wi-Fi:       Offline / Connecting / Online / Weak
TF Card:     Missing / Mounted / Error / Full
Backend:     Unknown / Online / Offline / Error
Market:      Idle / Fetching / Live / Stale / Error
Recording:   Idle / Recording / Saving / Saved / Failed
```

Market data freshness:

- `Live`: updated within expected time.
- `Stale`: past expected update window.
- `Offline`: network unavailable.
- `Error`: API failed.
- `Loading`: first load pending.

## Major Risks

- Scope sprawl: four apps can become four unfinished demos. Keep MVP local-first and shallow.
- Small screen complexity: 320x240 cannot hold mobile-style screens, long text, or complex charts.
- Binance connectivity: network, TLS, regional access, rate limits, and WebSocket reconnects need explicit states.
- API key leakage: AI keys and private exchange credentials must never enter firmware.
- Audio complexity: ES7210/ES8311 initialization, I2S DMA, WAV headers, and TF-card write jitter need isolated validation.
- TF card corruption: use simple file formats and safe write patterns.
- Long-running stability: watch memory, file handles, reconnect loops, and UI redraw pressure.

## Recommended Implementation Order

1. Local board bring-up: ESP-IDF, LCD, touch, Wi-Fi, TF card.
2. App shell: status bar, home dashboard, app navigation.
3. Market MVP: Binance REST watchlist, stale/error state, no trading UI.
4. Phrase MVP: local JSON cards and review state.
5. Recording MVP: ES7210 to WAV on TF card with duration and level.
6. Translation MVP: backend `/translate`, structured JSON, save as phrase card.
7. Backend sync: phrase sync and recording metadata.
8. Market upgrade: backend cache or WebSocket aggregation.
9. Geek controls: IMU gestures, speaker alerts, LAN debug page, themes.
