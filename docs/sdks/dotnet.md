# .NET SDK

The .NET SDK provides official libraries for integrating Flare into your .NET applications.

## Packages

The SDK ships as three NuGet packages — pick the one that fits your setup:

| Package | Description |
|---|---|
| `Flare.OpenFeature.Provider` | OpenFeature provider with caching and background polling. **Recommended.** |
| `Flare.Extensions.Configuration` | ASP.NET Core configuration provider — exposes flags via `IConfiguration` |
| `Flare.HttpClient` | Bare-bones HTTP client for direct Flare API access |

### Which package should I use?

**Start with `Flare.OpenFeature.Provider`** — it's the recommended way to use Flare in .NET. [OpenFeature](https://openfeature.dev) is a vendor-neutral standard for feature flags, so your application code stays decoupled from Flare. The provider handles caching and background polling out of the box.

Use **`Flare.Extensions.Configuration`** if you want to expose Flare flags through the standard `IConfiguration` interface — useful for legacy codebases or when you prefer the familiar `appsettings.json`-style access pattern. Requires .NET 10+.

Use **`Flare.HttpClient`** only if you need direct API access without any abstraction layer.

### .NET Version Support

| Package | Target frameworks |
|---|---|
| `Flare.HttpClient` | `netstandard2.0` → .NET 6, 7, 8, 9+, .NET Framework 4.6.1+ |
| `Flare.OpenFeature.Provider` | `netstandard2.0` → .NET 6, 7, 8, 9+, .NET Framework 4.6.1+ |
| `Flare.Extensions.Configuration` | `net10.0` → .NET 10+ only |

---

## Flare.OpenFeature.Provider

### Installation

```bash
dotnet add package Flare.OpenFeature.Provider
```

### Setup

Register the provider in your DI container. You have two options:

**Explicit options:**

```csharp
services.AddOpenFeature(builder =>
{
    builder.AddFlareProvider(new FlareApiClientOptions
    {
        BaseUrl = "https://flare.example.com",
        ApiKey = "your-api-key",
        Scope = "production"
    });
});
```

**From `IConfiguration`** (reads the `"Flare"` section by default):

```csharp
services.AddOpenFeature(builder =>
{
    builder.AddFlareProvider(configuration);

    // or with a custom section name:
    builder.AddFlareProvider(configuration, sectionName: "FlareSettings");
});
```

`appsettings.json` example:

```json
{
  "Flare": {
    "BaseUrl": "https://flare.example.com",
    "ApiKey": "your-api-key",
    "Scope": "production"
  }
}
```

### Configuration Options

**`FlareApiClientOptions`** (required):

| Option | Description |
|---|---|
| `BaseUrl` | URL of your Flare server |
| `ApiKey` | Project API key |
| `Scope` | Scope **alias** to evaluate flags in (e.g. `prod`, `staging`) — find it in **Project → Scopes** |

**`FlareProviderOptions`** (optional, passed as a second argument):

| Option | Default | Description |
|---|---|---|
| `CachingEnabled` | `true` | Load all flags on startup and serve from cache |
| `PollingInterval` | `30s` | How often the background poller refreshes the cache |
| `StaleThreshold` | `3` | Consecutive failures before a `ProviderStale` event is emitted |

```csharp
builder.AddFlareProvider(
    new FlareApiClientOptions { BaseUrl = "...", ApiKey = "...", Scope = "..." },
    new FlareProviderOptions
    {
        PollingInterval = TimeSpan.FromMinutes(1),
        CachingEnabled = true
    });
```

### Evaluating Flags

Inject `IFeatureClient` and use the standard OpenFeature API:

```csharp
public class MyService(IFeatureClient featureClient)
{
    public async Task DoSomethingAsync()
    {
        bool enabled = await featureClient.GetBooleanValueAsync("my-feature", defaultValue: false);
        string variant = await featureClient.GetStringValueAsync("theme", defaultValue: "default");
        int limit = await featureClient.GetIntegerValueAsync("rate-limit", defaultValue: 100);
    }
}
```

The second argument is the **default value** — returned if the flag doesn't exist, the provider is unavailable, or no rule matches.

### Evaluation Context

Pass an `EvaluationContext` to target specific users or apply rules based on attributes:

```csharp
var context = EvaluationContext.Builder()
    .SetTargetingKey("user-123")
    .Set("plan", "premium")
    .Set("region", "eu-west")
    .Build();

bool enabled = await featureClient.GetBooleanValueAsync("my-feature", false, context);
```

The `targetingKey` is matched against segment members. Additional attributes are matched against targeting rule conditions.

---

## Flare.Extensions.Configuration

### Installation

```bash
dotnet add package Flare.Extensions.Configuration
```

> Requires .NET 10+.

### Setup

```csharp
// Program.cs
builder.Configuration.AddFlareConfiguration();

builder.Services.AddFlareBackgroundService(options =>
{
    options.ServerUrl = "https://flare.example.com";
    options.ApiKey = "your-api-key";
    options.ScopeAlias = "production";
    options.ReloadInterval = TimeSpan.FromSeconds(60);
});
```

Or bind from `appsettings.json`:

```csharp
builder.Services.AddFlareBackgroundService(builder.Configuration, "Flare");
```

**Options:**

| Option | Default | Description |
|---|---|---|
| `ServerUrl` | — | URL of your Flare server |
| `ApiKey` | — | Project API key |
| `ScopeAlias` | — | Scope to evaluate flags in |
| `ReloadInterval` | `TimeSpan.Zero` | How often flags are refreshed. Must be set explicitly to enable polling. |
| `FeatureFlagSection` | `"FeatureFlags"` | The `IConfiguration` key prefix for flags |

### Reading Flags

Flags are stored under `{FeatureFlagSection}:{flagKey}`. With the default section:

```csharp
// Inject IConfiguration
bool enabled = configuration["FeatureFlags:my-feature"] == "true";
```

To change the section prefix:

```csharp
options.FeatureFlagSection = "Flags"; // → "Flags:my-feature"
```

For reactive updates when flags change without restarting the app, use `IOptionsMonitor<T>` bound to the configuration section.

---

## Flare.HttpClient

### Installation

```bash
dotnet add package Flare.HttpClient
```

### Setup

```csharp
services.AddFlareHttpClient(new FlareApiClientOptions
{
    BaseUrl = "https://flare.example.com",
    ApiKey = "your-api-key",
    Scope = "production"
});
```

Inject `IFlareApiClient` into your services.

### Evaluating Flags

**Single flag:**

```csharp
FlagEvaluationResponse result = await client.EvaluateAsync(
    "my-feature",
    new FlareEvaluationContext
    {
        Scope = "production",
        TargetingKey = "user-123",
        Attributes = new Dictionary<string, string?>
        {
            ["plan"] = "premium"
        }
    });
```

**All flags at once:**

```csharp
FlareEvaluateAllResponse all = await client.EvaluateAllAsync(
    new FlareEvaluationContext
    {
        Scope = "production",
        TargetingKey = "user-123"
    });
```

Use `EvaluateAllAsync` when you need all flags at startup — one request instead of N.
