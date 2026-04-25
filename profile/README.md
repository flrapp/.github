<div align="center">
  <img src="favicon.svg" width="80" alt="Flare" />
  <h1>Flare</h1>
  <p><em>Self-hosted feature flags. Your infrastructure, your rules.</em></p>
  <a href="https://opensource.org/licenses/MIT">
    <img src="https://img.shields.io/badge/License-MIT-FF9500.svg" alt="MIT License" />
  </a>
</div>

---

## What is Flare?

Flare is an open source, self-hosted feature flag management system. Create projects, define environment scopes (dev, staging, production), and manage feature flags per scope — with a granular permission system down to the scope level.

No third-party SaaS. No vendor lock-in. Full control.

---

## 🚧 Quick Start

<pre>
  / \__
 (    @\___    "Setup guide is coming, I promise! Woof!"
 /         O
/   (_____/
/_____/   U
</pre>

> Quick Start is under construction. Explore the repositories below in the meantime.

---

## Features

**Projects & Scopes**
- 🗂 Create isolated projects with unique aliases and API keys
- 🌍 Define custom scopes per project (dev / staging / production)
- 📦 Archive and unarchive projects

**Feature Flags**
- 🚩 Create flags with unique keys per project
- 🔀 Toggle flags independently per scope
- 🎯 Targeting rules with conditions (attribute-based, segment-based)
- 👥 Segments with explicit member lists
- ↕️ Rule priority and reordering

**User Management**
- 🔐 Admin-managed user accounts
- 🔒 Granular project-level permissions: flags, scopes, API keys, and more
- 🛡 Granular scope-level permissions: read and update flags per scope

**SDK Evaluation**
- ⚡ Single and bulk flag evaluation endpoints
- 🚦 Rate limiting per project (sliding window)

**Observability**
- 📡 OpenTelemetry tracing and metrics (OTLP)
- 📋 Audit log streams for project and user events
- 📊 Custom metrics per flag evaluation

---

## SDKs & Ecosystem

| Repository | Description |
|------------|-------------|
| [flare-api](https://github.com/flrapp/flare-api) | 🔧 Backend API |
| [flare-ui](https://github.com/flrapp/flare-ui) | 🖥 Admin Panel UI |
| [flare-dotnet](https://github.com/flrapp/flare-dotnet) | 🟣 .NET SDK (OpenFeature) |
| [flare-js](https://github.com/flrapp/flare-js) | 🟡 JavaScript / React SDK |

---

## 📚 Documentation

<pre>
  /\_____/\
 /  o   o  \    "I'll write the docs... when I feel like it."
( ==  ^  == )
 )         (
(  _______  )
(__(__)__(__)__)
</pre>

> Documentation is under construction.

---

## License

MIT
