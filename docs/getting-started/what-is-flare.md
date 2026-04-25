# What is Flare?

Flare is a self-hosted, open-source feature flag management system built as a free alternative to LaunchDarkly, Unleash, and Flagsmith — with **all features included, no paywalls, no usage limits**.

## Why Flare?

Most feature flag platforms charge for the capabilities teams actually need: per-seat pricing, limited environments, locked-behind-enterprise SSO or audit logs. Flare gives you the full feature set out of the box because you run it yourself.

| | LaunchDarkly | Unleash | Flagsmith | **Flare** |
|---|---|---|---|---|
| Self-hosted | ✅ (Enterprise) | ✅ | ✅ | ✅ |
| Fully free | ❌ | Partial | Partial | ✅ |
| OpenFeature compatible | ✅ | ✅ | ✅ | ✅ |
| All features on free tier | ❌ | ❌ | ❌ | ✅ |

## Core Capabilities

### Role-Based Access Control
Projects in Flare have granular permission management. Team members are assigned roles at the project level, so you control exactly who can create flags, toggle them in production, or only view results.

### Scopes and Environments
Each project supports multiple **scopes** — isolated environments like `development`, `staging`, and `production`. Flags are managed independently per scope, so a flag enabled in staging doesn't automatically go live in production.

### Flag Types
Flare supports multiple flag types to cover real-world use cases:

- **Boolean** — simple on/off toggle
- **String** — return a text value (e.g. a UI variant label)
- **Number** — return a numeric value (e.g. a rate limit or threshold)
- **JSON** — return structured configuration

### Evaluation Rules
Flags aren't just global switches. Flare lets you define **targeting rules** using an evaluation context — attributes passed at evaluation time (user ID, country, plan, device, etc.). Rules are evaluated in order, with a default fallback value when no rule matches.

### OpenFeature Compatible
Flare is built around the [OpenFeature](https://openfeature.dev/) standard. Your application code stays vendor-neutral — swap or extend providers without changing flag evaluation logic. Official SDKs implement the OpenFeature provider interface.

## Who is Flare for?

- Teams that want feature flags without handing data to a third party
- Organizations with compliance requirements that mandate on-premise tooling
- Startups and indie developers who need the full feature set without enterprise pricing
- Anyone who wants to own their feature management infrastructure completely
