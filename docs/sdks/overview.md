# SDKs Overview

Flare SDKs integrate flag evaluation into your application. All official SDKs implement the [OpenFeature](https://openfeature.dev) standard, so your application code stays vendor-neutral.

## Available SDKs

| Language / Platform | Package | OpenFeature |
|---|---|---|
| [.NET](./dotnet.md) | `Flare.OpenFeature.Provider` | ✅ |
| [JavaScript / React](./javascript-react.md) | `@flrapp/core` · `@flrapp/react` | ✅ |

## How Evaluation Works

SDKs authenticate with a **project API key** and send evaluation requests to the Flare API. The API applies targeting rules against the provided context and returns the resolved flag value along with the reason it was served.

```
Application                     Flare API
────────────────────────────────────────────────────────
evaluate("my-flag", context) →  match targeting rules
                              ←  { value, reason, variant }
```

### Evaluation Context

Every evaluation call accepts a context object:

```json
{
  "scope": "production",
  "targetingKey": "user-123",
  "attributes": {
    "plan": "premium",
    "country": "US"
  }
}
```

- **`scope`** — the scope **alias** to evaluate in (e.g. `prod`, `staging`). This is the alias shown in **Project → Scopes**, not the display name.
- **`targetingKey`** — unique identifier for the current user or entity
- **`attributes`** — arbitrary key-value pairs matched against targeting rule conditions

### Evaluation Response

Each flag evaluation returns:

| Field | Description |
|---|---|
| `value` | The resolved flag value |
| `variant` | The variant name (rule name or `"default"`) |
| `reason` | Why this value was returned (`TARGETING_MATCH`, `DEFAULT`, `ERROR`, etc.) |
| `type` | The flag type (`boolean`, `string`, `number`, `json`) |

## Caching and Polling

The recommended SDKs (`Flare.OpenFeature.Provider`, `@flrapp/core`) load all flags at startup and keep them in a local cache. A background poller refreshes the cache on a configurable interval (default: 30 seconds).

This means:
- Flag evaluations are served from memory — no network latency per evaluation
- There is a propagation delay equal to the polling interval when flags are changed in the UI
- If the Flare API is temporarily unreachable, the last cached values continue to be served

## Choosing an Integration Pattern

| If you need... | Use |
|---|---|
| Standard OpenFeature API | `Flare.OpenFeature.Provider` (.NET) or `@flrapp/core` (JS) |
| Flags via `IConfiguration` (legacy .NET code) | `Flare.Extensions.Configuration` |
| React hooks (`useFlag`) | `@flrapp/react` |
| Direct HTTP access / custom integration | `Flare.HttpClient` or the raw [Flag Evaluation API](../api-reference/flag-evaluation.md) |
