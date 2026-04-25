# Targeting Rules & Segments

Targeting rules let you return different flag values for different users — without changing the default value or deploying new code.

---

## How Evaluation Works

When the SDK evaluates a flag, Flare processes rules in this order:

1. Check targeting rules **top to bottom** (priority order)
2. For each rule, check all its **conditions** — all must pass (AND logic)
3. If a rule matches → return its **serve value** and stop
4. If no rule matches → return the **default value**

---

## Targeting Rules

### Creating a Rule

Open a flag, select a scope, and click **Add Rule**.

Each rule requires:

| Field | Description |
|---|---|
| Serve value | What to return when this rule matches (same type as the flag) |
| Conditions | One or more attribute checks — all must be true for the rule to fire |

### Rule Priority

Rules are evaluated in the order they appear in the list. The **first matching rule wins** — subsequent rules are not evaluated.

You can reorder rules by dragging them or using the reorder action. Adjust order carefully: a broad rule placed above a narrow one will always match first, making the narrow rule unreachable.

### Example

Flag: `show-discount-banner` (boolean), scope: `production`

| Priority | Condition | Serve value |
|---|---|---|
| 1 | `plan` equals `enterprise` | `false` |
| 2 | `country` in `[US, CA]` | `true` |
| Default | — | `false` |

Enterprise users never see the banner. US and Canadian non-enterprise users do. Everyone else does not.

---

## Conditions

A condition is a single comparison: `attributeKey operator value`.

Attributes come from the evaluation context passed by the SDK at runtime.

### Supported Operators

| Operator | Description |
|---|---|
| `equals` | Exact match |
| `not equals` | Does not match |
| `contains` | String contains substring |
| `not contains` | String does not contain substring |
| `in` | Value is in a list |
| `not in` | Value is not in a list |
| `greater than` | Numeric comparison |
| `less than` | Numeric comparison |

### Multiple Conditions in a Rule

All conditions within a rule use **AND logic** — every condition must be true for the rule to match.

To express OR logic, create separate rules with the same serve value.

```
Rule A: country = US  AND  plan = pro   → serve true
Rule B: country = CA  AND  plan = pro   → serve true
```

---

## Segments

A **segment** is a saved list of users identified by their `targetingKey`. Use segments to manage user groups in one place and reference them across multiple rules and flags.

### Creating a Segment

Go to **Project → Segments → New Segment** and provide a name and optional description.

### Adding Members

After creating a segment, add members by their `targetingKey` values:

```
user-123
user-456
service-account-webhook
```

You can add multiple keys at once. Members can be removed individually at any time.

### Using Segments in Rules

To target a segment, add a condition to a rule with these values:

| Field | Value |
|---|---|
| Attribute key | `targetingKey` |
| Operator | `in` |
| Value | the segment name (e.g. `beta-testers`) |

Flare resolves segment membership at evaluation time by matching the `targetingKey` from the evaluation context against the segment's member list.

This means you can add or remove users from a segment without touching the flag or its rules — the change takes effect immediately on the next evaluation.

### Segment Lifecycle

- **Rename** a segment at any time — rules that reference it remain valid
- **Delete** a segment — removes it and any conditions that reference it; review affected rules before deleting
