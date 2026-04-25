# Authentication

The Flare SDK API uses API keys to authenticate requests. Each project has a unique API key that you can find (or regenerate) in the project settings.

> The SDK API (`/sdk/v1/...`) uses API key authentication. The management UI uses a separate session-based mechanism — the API key grants no access to management endpoints.

## How to Authenticate

Pass your API key as a Bearer token in the `Authorization` header:

```
Authorization: Bearer <your-api-key>
```

### Example

```bash
curl -X POST https://your-flare-instance/sdk/v1/flags/evaluate \
  -H "Authorization: Bearer flr_a1b2c3d4e5f6..." \
  -H "Content-Type: application/json" \
  -d '{
    "flagKey": "my-feature",
    "context": {
      "scope": "production",
      "targetingKey": "user-123"
    }
  }'
```

## Error Responses

| Status | Reason |
|---|---|
| `401` | Missing `Authorization` header, wrong scheme, or invalid API key |

## Getting Your API Key

1. Open your project in the Flare dashboard
2. Go to **Settings → API Key**
3. Copy the key, or click **Regenerate** to issue a new one

> **Warning:** Regenerating the key immediately invalidates the previous one. Update all SDK integrations before regenerating in production.
