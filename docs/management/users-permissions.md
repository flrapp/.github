# Users & Permissions

## User Accounts

Flare uses a simple user model: accounts are created by a system administrator and exist globally across all projects.

### Creating a User

Only admins can create users. Go to **Admin → Users → New User** and provide:

| Field | Description |
|---|---|
| Username | Login identifier |
| Full name | Display name shown in the UI |
| Temporary password | User must change this on first login |

New users are created with `mustChangePassword = true`. On first login they are prompted to set a permanent password before accessing anything else.

### Managing Users

From the **Admin → Users** list you can:

| Action | Description |
|---|---|
| Edit | Change full name or global role |
| Deactivate | Prevent login without deleting the account |
| Activate | Re-enable a deactivated account |
| Reset password | Issue a new temporary password |
| Unlock | Remove a brute-force lockout (see below) |
| Delete | Permanently remove the account |

### Brute-Force Protection

After a configurable number of failed login attempts, an account is automatically locked (`isBruteForceLocked`). The user cannot log in until an admin unlocks the account from the user management page.

---

## Global Roles

Every user has a **global role** that controls system-level access:

| Role | Capabilities |
|---|---|
| **Admin** | Full access: manage users, create/delete projects, access all settings |
| **User** | Can only access projects they've been explicitly added to |

Admins can change any user's global role at any time.

---

## Project Permissions

Being added to a project grants access based on the permissions assigned. Project permissions control what a user can do at the project level:

| Permission | Description |
|---|---|
| View project | See the project and its flags (read-only) |
| Manage flags | Create, edit, and delete feature flags |
| Manage scopes | Create and delete scopes |
| Manage segments | Create, edit, and delete user segments |
| Manage team | Add/remove members, update permissions |

---

## Scope Permissions

Scope permissions are granted **per scope** within a project. This means a user can have different access levels in different environments.

| Permission | Description |
|---|---|
| View | See flag values and targeting rules in this scope |
| Update flag values | Change the default value of a flag in this scope |
| Manage targeting rules | Add, edit, delete, and reorder targeting rules |

### Example Permission Setup

| User | Project permissions | `development` | `staging` | `production` |
|---|---|---|---|---|
| Lead engineer | Manage flags, manage team | Full | Full | Full |
| Developer | Manage flags | Full | Full | View only |
| QA | View project | View | Full | View only |

This ensures `production` changes require elevated access while developers have freedom in lower environments.

---

## My Permissions

Users can view their own effective permissions for any project via **Project → My Permissions**. This is useful to understand what actions are available without trial and error.

---

## Session Management

- **Login** — returns a session with user details and global role
- **Change password** — required on first login when `mustChangePassword` is true
- **Logout** — invalidates the current session

Sessions are managed server-side. There is no token refresh flow — after expiry the user is redirected to the login screen.
