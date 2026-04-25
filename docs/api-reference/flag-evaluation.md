# Flag Evaluation

The SDK evaluation endpoints are the only Flare API endpoints your application talks to directly. All requests require a [project API key](./authentication.md).

Base path: `/sdk/v1`

---

## Evaluate a Single Flag

Resolves one flag for the given context.

```
POST /sdk/v1/flags/evaluate
```

### Request

```json
{
  "flagKey": "my-feature",
  "context": {
    "scope": "production",
    "targetingKey": "user-123",
    "attributes": {
      "plan": "pro",
      "country": "US"
    }
  }
}
```

| Field | Type | Required | Description |
|---|---|---|---|
| `flagKey` | string | ✅ | The flag's key as defined in the Flare dashboard |
| `context.scope` | string | ✅ | Scope **alias** (e.g. `dev`, `staging`, `prod`) — not the display name. Find the alias in **Project → Scopes**. |
| `context.targetingKey` | string | ❌ | Unique user / session / device identifier used in targeting rules |
| `context.attributes` | `map<string, string>` | ❌ | Custom attributes matched against targeting rule conditions |

### Response

```json
{
  "flagKey": "my-feature",
  "value": true,
  "variant": "on",
  "reason": "TARGETING_MATCH",
  "type": "boolean"
}
```

| Field | Description |
|---|---|
| `flagKey` | Echo of the requested flag key |
| `value` | Resolved flag value (`boolean`, `string`, `number`, or `json`) |
| `variant` | Name of the matched rule, or `"default"` if the default value was served |
| `reason` | Why this value was returned (see [Evaluation Reasons](#evaluation-reasons)) |
| `type` | Flag type: `boolean`, `string`, `number`, `json` |

### Error Responses

| Status | Reason |
|---|---|
| `401` | Invalid or missing API key |
| `404` | Flag key not found in the specified scope |

---

## Evaluate All Flags

Resolves all flags in the project for the given context in a single request. Use this at application startup to pre-populate a local cache.

```
POST /sdk/v1/flags/evaluate-all
```

### Request

```json
{
  "context": {
    "scope": "production",
    "targetingKey": "user-123",
    "attributes": {
      "plan": "pro"
    }
  }
}
```

The `context` object follows the same schema as the single-flag endpoint.

### Response

```json
{
  "flags": [
    {
      "flagKey": "my-feature",
      "value": true,
      "reason": "TARGETING_MATCH",
      "type": "boolean"
    },
    {
      "flagKey": "theme",
      "value": "dark",
      "reason": "DEFAULT",
      "type": "string"
    }
  ]
}
```

Each item in `flags` has the same fields as the single-flag response, **except `variant`** — it is not included in the bulk response.

If the project has no flags, the response is `{ "flags": [] }` — not an error.

### Error Responses

| Status | Reason |
|---|---|
| `401` | Invalid or missing API key |

---

## Evaluation Reasons

| Reason | Description |
|---|---|
| `TARGETING_MATCH` | A targeting rule matched the provided context |
| `DEFAULT` | No targeting rule matched; the default flag value was returned |
| `ERROR` | Evaluation failed (flag not found, invalid context, server error) |
