# Flare

Flare is an open source, self-hosted feature flag management system. It gives teams full control over feature rollouts across multiple environments — without relying on third-party SaaS.

## What is Flare?

Flare lets you create projects, define environment scopes (e.g. dev, staging, production), and manage boolean feature flags per scope. Access to every action is controlled by a granular permission system — down to the scope level — so teams can safely delegate flag management without granting broad access.

SDKs are available for .NET (via OpenFeature) and JavaScript/React, with more planned.

---

## Current Features

**Projects & Scopes**
- Create isolated projects, each with a unique alias and API key
- Define custom scopes per project to represent environments
- Archive and unarchive projects

**Feature Flags**
- Create flags with a unique key per project
- Toggle flags independently per scope
- Targeting rules with conditions (attribute-based, segment-based)
- Segments with explicit member lists
- Rule priority and reordering

**User Management**
- Admin-managed user accounts
- Granular project-level permissions: manage users, flags, scopes, API key access, and more
- Granular scope-level permissions: read and update flags per scope

**SDK Evaluation**
- Single and bulk flag evaluation endpoints
- Rate limiting on evaluation endpoints (per-project sliding window)

**Observability**
- OpenTelemetry tracing and metrics (OTLP) with OTel Collector
- Audit log streams for project and user events
- Custom metrics per flag evaluation (project / flag / scope tags)

**Operations**
- Docker Compose deployment
- Multi-replica safe: advisory locks for migrations, shared Data Protection key

---

## Roadmap

- [ ] Percentage rollout / gradual releases
- [ ] String, number, and JSON flag value types (full OpenFeature compliance)
- [ ] Read-through caching for high-throughput evaluation
- [ ] Primary/replica database support
- [ ] More SDK providers


## License

MIT
