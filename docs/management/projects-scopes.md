# Projects & Scopes

## Projects

A project is the top-level container for all your flags, scopes, segments, and team members. Each project gets its own API key for SDK integration.

### Creating a Project

In the UI, click **New Project** and fill in:

| Field | Description |
|---|---|
| Name | Human-readable label |
| Alias | Short identifier used in URLs (e.g. `my-app`) |
| Description | Optional context for teammates |

### Project Settings

From the project settings page you can:

- **Rename** the project
- **Get the API key** — used by SDKs to evaluate flags. Keep this secret.
- **Rotate the API key** — immediately invalidates the old key. All SDK instances must be updated.
- **Archive / Unarchive** — archived projects are hidden from the main list but not deleted. Useful for sunset products.
- **Delete** — permanently removes the project and all its data.

### API Key

The API key identifies your project to the SDK evaluation endpoints. It is separate from user credentials and has no management permissions — it can only evaluate flags.

Rotate the key if it is ever exposed. There is no grace period: the old key stops working immediately.

---

## Scopes

A scope is an isolated environment within a project. You define the scopes that match your deployment pipeline — common examples are `development`, `staging`, and `production`.

### Creating a Scope

From the project's **Scopes** tab, click **New Scope** and provide a name, alias, and optional description.

When a new scope is created, every existing flag in the project automatically gets an empty flag value for that scope. No manual setup required.

### Managing Scopes

- **Rename** a scope at any time — the alias remains stable and is what SDKs reference.
- **Delete** a scope — removes all flag values and targeting rules associated with it. This cannot be undone.

### Scope Isolation

Changes to a flag's value or targeting rules in one scope have **no effect** on other scopes. This means you can:

- Enable a flag in `staging` while it remains off in `production`
- Set different default values per environment
- Have different targeting rules per environment

SDK requests always specify a scope in the evaluation context, so evaluation is always scoped to exactly one environment.

---

## Team Management

Each project has its own team. Users must be explicitly added to a project before they can access it.

### Adding a Team Member

1. Go to **Project → Team**
2. Click **Add Member** — only users not already in the project are shown
3. Assign **project permissions** and **scope permissions** per scope

### Permissions Model

Permissions operate at two levels:

**Project permissions** control access to project-wide resources:
- View the project
- Manage flags (create, edit, delete)
- Manage scopes
- Manage team members
- Manage segments

**Scope permissions** are assigned individually per scope:
- View flag values in this scope
- Update flag values (default value)
- Manage targeting rules

This lets you give a developer full write access to `development` and `staging` while restricting `production` to a senior engineer or an automated deployment process.

### Updating or Removing Members

From the **Team** tab you can update a member's permissions at any time or remove them from the project entirely. Removing a member does not delete their user account.
