# JavaScript / React SDK

The Flare JavaScript SDK is built on the [OpenFeature](https://openfeature.dev) standard — a vendor-neutral specification for feature flag evaluation. It provides two packages:

| Package | Use case |
|---|---|
| `@flrapp/core` | Any JavaScript / TypeScript environment |
| `@flrapp/react` | React applications |

---

## Installation

```bash
# Core (framework-agnostic)
npm install @flrapp/core @openfeature/core

# React
npm install @flrapp/react @openfeature/react-sdk
```

---

## Configuration

Create a `FlareProvider` pointing to your self-hosted Flare instance:

```ts
import { FlareProvider } from '@flrapp/core';

const provider = new FlareProvider({
  baseUrl: 'https://your-flare-instance.com',
  apiKey: 'YOUR_API_KEY',
  scope: 'prod',
});
```

| Option | Required | Description |
|---|---|---|
| `baseUrl` | ✅ | URL of your Flare instance |
| `apiKey` | ✅ | Project API key |
| `scope` | ✅ | Environment defined in your Flare project (e.g. `dev`, `stage`, `prod`) |
| `pollingIntervalMs` | ❌ | Re-fetch all flags every N milliseconds |
| `timeout` | ❌ | HTTP request timeout in ms |

---

## Evaluation Context

The evaluation context tells Flare who is requesting the flag, enabling targeting rules to apply. Pass it when initializing the provider, or update it later when the user's state changes (e.g. after login).

```ts
const context = {
  targetingKey: 'user-123',  // unique user / session / device identifier
  plan: 'pro',               // arbitrary attributes your targeting rules reference
  region: 'eu-west',
};
```

- **`targetingKey`** — the primary identifier matched against segment members and targeting rule conditions
- **Additional attributes** — any string key-value pairs referenced in your Flare targeting rules

> **Note on context structure:** The OpenFeature JavaScript standard uses a flat context object — custom attributes are placed at the top level alongside `targetingKey`, not nested under an `attributes` key. The Flare provider maps this to the API format automatically. If you're looking at the raw [Flag Evaluation API](../api-reference/flag-evaluation.md), you'll see attributes nested under `"attributes": {}` — that's the wire format, not the SDK format.

---

## Usage — Core

```ts
import { OpenFeature } from '@openfeature/core';
import { FlareProvider } from '@flrapp/core';

// Initialize — fetches all flags for the given scope and context
await OpenFeature.setProviderAndWait(
  new FlareProvider({ baseUrl: '...', apiKey: '...', scope: 'prod' }),
  { targetingKey: 'user-123', plan: 'pro' }
);

const client = OpenFeature.getClient();

// Evaluate flags (resolved from local cache — no extra HTTP calls)
const isEnabled = await client.getBooleanValue('my-feature', false);
const theme     = await client.getStringValue('theme', 'default');
const rateLimit = await client.getNumberValue('rate-limit', 100);
```

On initialization the SDK fetches all flags for the scope in a single request and caches them locally. Individual evaluations are served from cache — no network latency per call.

---

## Usage — React

Wrap your application with `OpenFeatureProvider` and use the `useFlag` hook to read flag values in components:

```tsx
import { OpenFeatureProvider } from '@openfeature/react-sdk';
import { FlareProvider } from '@flrapp/core';
import { useFlag } from '@flrapp/react';

const provider = new FlareProvider({
  baseUrl: 'https://your-flare-instance.com',
  apiKey: 'YOUR_API_KEY',
  scope: 'prod',
});

function App() {
  return (
    <OpenFeatureProvider
      provider={provider}
      context={{ targetingKey: 'user-123', plan: 'pro' }}
    >
      <MyComponent />
    </OpenFeatureProvider>
  );
}

function MyComponent() {
  const { value } = useFlag('my-feature', false);
  return <>{value && <NewFeature />}</>;
}
```

`useFlag(flagKey, defaultValue)` returns an object with:
- `value` — the resolved flag value
- `reason` — why this value was returned (`TARGETING_MATCH`, `DEFAULT`, `ERROR`, etc.)

### Updating Context at Runtime

When the user logs in or their attributes change, update the global context. The SDK re-fetches flags automatically:

```ts
import { OpenFeature } from '@openfeature/core';

async function onLogin(user) {
  await OpenFeature.setContext({
    targetingKey: user.id,
    plan: user.plan,
  });
}
```

After `setContext` resolves, all `useFlag` hooks re-render with the new values.

---

## Polling

If `pollingIntervalMs` is set, the SDK re-fetches all flags in the background at that interval. This keeps the local cache in sync with changes made in the Flare UI without requiring a page reload.

```ts
const provider = new FlareProvider({
  baseUrl: '...',
  apiKey: '...',
  scope: 'prod',
  pollingIntervalMs: 30_000, // refresh every 30 seconds
});
```

Without polling, flags are fetched once on initialization and on each `setContext` call.
