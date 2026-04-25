# Audit Logs

Flare records a structured audit trail of all significant actions in the system. Audit events are emitted as structured log entries via OpenTelemetry and flow through the same pipeline as application logs.

To receive audit logs, configure the [OTel Collector](./opentelemetry.md) with a logs pipeline that exports to your log storage backend (e.g. Loki).

> Pre-built Grafana dashboards for audit logs are currently in development.

---

## Source Contexts

Audit events are split into two source contexts, which you can filter and route independently:

| SourceContext | Scope |
|---|---|
| `Flare.Audit.Project` | All actions within a project (flags, scopes, rules, segments, members) |
| `Flare.Audit.User` | System-level user account actions |

---

## Flare.Audit.Project

Each record contains: `Action`, `EntityType`, `Username`, `ProjectAlias`, `Scope`.  
Mutating operations additionally include `OldValue` and `NewValue`.

### Project

| Action | OldValue / NewValue |
|---|---|
| `Created` | — |
| `Updated` | ✅ |
| `Deleted` | — |
| `ApiKeyRegenerated` | — |
| `Archived` | — |
| `Unarchived` | — |

### Scope

| Action | OldValue / NewValue |
|---|---|
| `Created` | — |
| `Updated` | — |
| `Deleted` | — |

### FeatureFlag

| Action | OldValue / NewValue |
|---|---|
| `Created` | — |
| `Updated` | — |
| `Deleted` | — |
| `ValueUpdated` | ✅ |

### TargetingRule

| Action | OldValue / NewValue |
|---|---|
| `Created` | — |
| `Updated` | — |
| `Deleted` | — |
| `Reordered` | — |

### TargetingCondition

| Action | OldValue / NewValue |
|---|---|
| `Created` | — |
| `Updated` | — |
| `Deleted` | — |

### Segment

| Action | OldValue / NewValue |
|---|---|
| `Created` | — |
| `Updated` | — |
| `Deleted` | — |

### SegmentMember

| Action | OldValue / NewValue |
|---|---|
| `Added` | — |
| `Deleted` | — |

### ProjectMember

| Action | OldValue / NewValue |
|---|---|
| `UserInvited` | — |
| `UserRemoved` | — |
| `ProjectPermissionsAssigned` | — |
| `ProjectPermissionRevoked` | — |
| `ScopePermissionsAssigned` | — |
| `ScopePermissionRevoked` | — |
| `PermissionsUpdated` | ✅ |

---

## Flare.Audit.User

Each record contains: `Action`, `EntityType`, `SubjectUsername` (who the action affects), `ActorUsername` (who performed it), `Scope`.  
For self-service operations (login, password change) `SubjectUsername == ActorUsername`.  
Mutating operations additionally include `OldValue` and `NewValue`.

### User

| Action | Initiated by | OldValue / NewValue |
|---|---|---|
| `LoggedIn` | User themselves | — |
| `LoggedOut` | User themselves | — |
| `PasswordChanged` | User themselves | — |
| `Created` | Administrator | — |
| `Updated` | Administrator | ✅ |
| `PasswordReset` | Administrator | — |
| `Activated` | Administrator | — |
| `Deactivated` | Administrator | — |
| `HardDeleted` | Administrator | — |
| `BruteForceUnlocked` | Administrator | — |

---

## Structured Properties

All audit fields are indexed as structured properties and can be filtered and grouped in any combination.

| Property | Present in | Example value |
|---|---|---|
| `Action` | Project + User | `ValueUpdated`, `LoggedIn` |
| `EntityType` | Project + User | `FeatureFlag`, `Segment`, `User` |
| `ProjectAlias` | Project | `my-app` |
| `Scope` | Project (scope-aware events) | `production`, `dev` |
| `Username` | Project | `john.doe` |
| `SubjectUsername` | User | `john.doe` |
| `ActorUsername` | User | `admin` |
| `OldValue.*` | Project + User (mutations) | Any field of the changed object |
| `NewValue.*` | Project + User (mutations) | Any field of the changed object |

### OldValue / NewValue format

`OldValue` and `NewValue` are flattened into dot-notation properties. For example, a `FeatureFlag` `ValueUpdated` event where the default value changed from `false` to `true` looks like this:

```json
{
  "Action": "ValueUpdated",
  "EntityType": "FeatureFlag",
  "ProjectAlias": "my-app",
  "Scope": "production",
  "Username": "john.doe",
  "OldValue.DefaultValue": "false",
  "NewValue.DefaultValue": "true"
}
```

A `User` `Updated` event (admin changed a user's role):

```json
{
  "Action": "Updated",
  "EntityType": "User",
  "SubjectUsername": "john.doe",
  "ActorUsername": "admin",
  "OldValue.GlobalRole": "User",
  "NewValue.GlobalRole": "Admin"
}
```

To route by domain, filter on `SourceContext`:
- `Flare.Audit.Project` — everything related to projects
- `Flare.Audit.User` — system-level account actions

---

## Querying in Loki

Filter all project audit events:

```logql
{service_name="flare-api"} | json | SourceContext = "Flare.Audit.Project"
```

Filter by entity type:

```logql
{service_name="flare-api"} | json
  | SourceContext = "Flare.Audit.Project"
  | EntityType = "FeatureFlag"
```

Filter by actor:

```logql
{service_name="flare-api"} | json
  | SourceContext = "Flare.Audit.User"
  | ActorUsername = "admin"
```

Filter by project and environment:

```logql
{service_name="flare-api"} | json
  | SourceContext = "Flare.Audit.Project"
  | ProjectAlias = "my-app"
  | Scope = "production"
```
