# Core Concepts

Understanding a handful of key concepts will make everything else in Flare click into place.

## Entity Hierarchy

```
System
├── Users              ← global accounts, managed by admins
└── Projects           ← shared resources; users are added as members
    ├── Scopes          ← isolated environments (prod, staging, dev)
    ├── Segments        ← named groups of users
    └── Feature Flags
        └── Flag Value  ← one per Scope
            └── Targeting Rules
                └── Conditions
```

Projects are not owned by a single user — they are team resources. Any user can be added to a project with specific permissions.

---

## Projects

A **project** is the top-level container for your feature flags. It maps to one product, service, or application.

Each project has:
- Its own set of **scopes**, **flags**, and **segments**
- A dedicated **API key** used by SDKs to evaluate flags
- A team of users with individual permissions

The API key is separate from user credentials and is the only authentication method accepted by the SDK evaluation endpoints.

---

## Scopes

A **scope** is an isolated environment within a project — typically `production`, `staging`, and `development`, but you can create any scopes that fit your workflow.

Each scope has a **name** (display label) and an **alias** (short identifier, e.g. `prod`, `dev`). The alias is what you pass in the evaluation context when calling the SDK or API — it never changes even if you rename the scope.

Key behavior:
- Every flag has a **separate value per scope**. Enabling a flag in `staging` has no effect on `production`.
- When a new scope is created, all existing flags automatically get a default (empty) value for it.
- SDK requests always target a specific scope via the evaluation context.

---

## Feature Flags

A **feature flag** is a named toggle with a type. When you create a flag, you choose one of four types:

| Type | Example use case |
|---|---|
| `boolean` | Enable/disable a feature |
| `string` | A/B test variant label, config string |
| `number` | Rate limit value, threshold, percentage |
| `json` | Structured configuration object |

The flag's **key** (e.g. `enable-new-checkout`) is what SDKs reference at evaluation time. The key must be unique within a project.

---

## Flag Values

A **flag value** is the concrete value of a flag within a specific scope. Each flag has exactly one flag value per scope.

A flag value consists of:
- A **default value** — returned when no targeting rule matches
- Zero or more **targeting rules** — evaluated in priority order before the default

---

## Targeting Rules

A **targeting rule** defines what value to serve when a set of conditions is satisfied.

Each rule has:
- **Conditions** — one or more attribute checks that must all pass (AND logic)
- **Serve value** — the flag value to return if the rule matches
- **Priority** — rules are evaluated top-to-bottom; the first match wins

If no rule matches, the default flag value is returned.

### Conditions

A condition compares an attribute from the evaluation context against a fixed value using an operator:

```
attributeKey  operator  value
───────────────────────────────
country       equals    US
plan          in        [pro, enterprise]
beta_user     equals    true
```

Supported operators include: `equals`, `not equals`, `contains`, `not contains`, `in`, `not in`, `greater than`, `less than`, and others.

---

## Segments

A **segment** is a named list of users identified by their `targetingKey`. Segments let you group users once and reuse that group across multiple targeting rules.

Example: a `beta-testers` segment containing `["user-123", "user-456"]` can be referenced in a condition instead of listing individual keys each time.

---

## Evaluation Context

When your application evaluates a flag, it passes an **evaluation context** alongside the request:

```json
{
  "scope": "production",
  "targetingKey": "user-123",
  "attributes": {
    "country": "US",
    "plan": "pro",
    "beta_user": true
  }
}
```

| Field | Description |
|---|---|
| `scope` | Which scope to evaluate the flag in |
| `targetingKey` | Unique identifier for the current user or entity |
| `attributes` | Arbitrary key-value pairs used in targeting conditions |

The API matches the `targetingKey` against segment members and evaluates each targeting rule's conditions against the provided attributes.

---

## Roles and Permissions

Flare has two layers of access control:

### Global Role
Applies system-wide. Admins can manage all users, projects, and system settings. Regular users can only access projects they've been added to.

### Project & Scope Permissions
Each project member has explicit permissions at two levels:

- **Project permissions** — e.g. manage flags, manage team members, view the project
- **Scope permissions** — granted per scope, e.g. toggle flags in `production` but only view in `development`

This lets you give a developer full control over `staging` while restricting write access to `production` to a smaller group.

---

## SDK Authentication

SDKs authenticate using a **project API key**, not user credentials. The API key is scoped to a single project and is used only for flag evaluation — it cannot modify flags or manage the project.

You can rotate the API key at any time from the project settings. After rotation, the old key is immediately invalidated.

---

## Putting It Together

A typical setup looks like this:

1. Create a project → get the API key
2. Create scopes: `development`, `staging`, `production`
3. Create a flag (e.g. `enable-new-checkout`, type `boolean`)
4. Set the default value per scope (e.g. `false` everywhere)
5. Add a targeting rule in `staging`: serve `true` when `plan in [pro]`
6. Use the SDK to evaluate `enable-new-checkout` — it returns `true` for pro users in staging, `false` for everyone else
