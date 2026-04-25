# Feature Flags

## Creating a Flag

In your project, go to **Flags → New Flag** and provide:

| Field | Description |
|---|---|
| Name | Human-readable label shown in the UI |
| Key | Identifier used in SDK calls (e.g. `enable-new-checkout`) |
| Type | `boolean`, `string`, `number`, or `json` |
| Description | Optional context for the team |

The **key** must be unique within the project. Choose it carefully — changing the key later requires updating all SDK call sites.

### Flag Types

| Type | Default value | Typical use |
|---|---|---|
| `boolean` | `false` | Feature on/off toggle |
| `string` | `""` | Variant label, config string, theme name |
| `number` | `0` | Rate limit, threshold, percentage rollout value |
| `json` | `null` | Structured config, multi-property feature settings |

---

## Flag Values per Scope

When a flag is created, Flare automatically creates a **flag value** for every scope in the project. Each flag value starts with an empty/default state.

To configure a flag in a specific scope:

1. Open the flag
2. Select the scope tab (e.g. `production`)
3. Set the **default value** — returned when no targeting rule matches
4. Optionally add **targeting rules** (see [Targeting Rules & Segments](./targeting-rules-segments.md))

Flag values in different scopes are completely independent. The `production` value is never affected by changes to `staging`.

---

## Editing a Flag

From the flag detail page you can update the flag's **name**, **key**, and **description**.

> Changing the key of an active flag will break all SDK integrations that use the old key. Update your application code before or immediately after renaming.

---

## Deleting a Flag

Deleting a flag removes it and all its values, targeting rules, and conditions across every scope. This cannot be undone.

Before deleting, ensure the flag is no longer referenced in your application code. SDKs evaluating a deleted flag will receive the default "flag not found" response per the OpenFeature specification.

---

## Flag Lifecycle

A typical flag goes through these stages:

```
Create → Configure default values → Add targeting rules → Ship → Clean up
```

1. **Create** the flag with a safe default (usually `false` for booleans)
2. **Configure** values per scope — enable in `development` first
3. **Add targeting rules** to gradually roll out (specific users, segments, attributes)
4. **Promote** to `staging` and then `production` by updating those scope values
5. **Clean up** — once fully rolled out and stable, remove the flag from code and delete it from Flare
