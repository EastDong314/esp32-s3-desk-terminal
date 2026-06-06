# Repository Instructions

## Scope

These instructions apply to the entire repository.

## Development Guidance

- Keep changes focused on the ESP32-S3 desk terminal project.
- Prefer ESP-IDF-compatible code and examples unless the project later commits to another framework.
- Do not place secrets, API keys, exchange credentials, or OpenAI keys in firmware source files.
- Use a backend proxy for AI translation requests.
- Treat Binance functionality as display-only unless trading support is explicitly requested later.
- Keep local data formats simple and documented, preferably JSON for phrase cards and app settings.
- Use TF card storage for recordings, translation history, and phrase data where practical.

## Suggested Structure

```text
firmware/
  main/
  components/
backend/
docs/
data/
```

## Verification

- Document any hardware examples used from the LCKFB wiki.
- For firmware changes, note whether the code has been built, flashed, or only statically reviewed.
- For backend changes, include a simple local smoke test where possible.
