# dictate-models

Remote model catalog for [Dictate](https://github.com/ozangencer/Dictate) — a native macOS menu bar speech-to-text app.

The Dictate app fetches `manifest.json` from this repo on launch (and on demand via the Settings "Refresh catalog" button) to decide which local Whisper models to offer in the Local Models tab. This lets the curated list evolve without shipping a new app build.

## Schema

```json
{
  "schemaVersion": 1,
  "version": "<YYYY-MM-DD.N>",
  "models": [
    {
      "id": "<whisperkit variant string, e.g. openai_whisper-large-v3-turbo>",
      "engine": "whisperkit",
      "displayName": "Large v3 Turbo",
      "description": "Short blurb shown under the name.",
      "sizeMB": 1580,
      "quality": 5,
      "speed": 4,
      "languages": ["multi"],
      "recommended": true
    }
  ]
}
```

### Fields

| Field | Type | Notes |
|---|---|---|
| `id` | string | Must match the WhisperKit variant the app passes to `WhisperKit(model:)`. |
| `engine` | string | Currently only `whisperkit`. Future engines (parakeet-mlx, moonshine) will extend this. |
| `displayName` | string | Human-readable name. |
| `description` | string | One-line blurb. |
| `sizeMB` | int | Approximate download size in MB. Shown to the user. |
| `quality` | int 1–5 | Star rating shown in UI. |
| `speed` | int 1–5 | Star rating shown in UI. |
| `languages` | string[] | Use `["multi"]` for multilingual, otherwise ISO codes. |
| `recommended` | bool | If true, appears in the top "Recommended" section of the app. |

### Bumping `version`

`version` is a free-form string; the app only displays it ("Catalog v2026-04-13.1"). Bump it whenever you change the model list so users can verify what they're on.

### `schemaVersion`

The app refuses to load a manifest whose `schemaVersion` it doesn't understand — it falls back to its cached or bundled copy. Only increment this when you introduce a breaking change to the schema.

## How the app consumes it

1. On launch, the app tries to fetch this `manifest.json` from `raw.githubusercontent.com/ozangencer/dictate-models/main/manifest.json`.
2. On success, it caches the JSON at `~/Library/Application Support/Dictate/models-manifest.json`.
3. On failure (offline / 404 / network), it uses the cached copy, then falls back to a manifest bundled inside the app binary.
4. The user can force a refresh from Settings → Local Models → ↻ button.
