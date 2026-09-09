---
title: "drop_fields"
date: 0001-01-01
summary: "drop_fields #     Category Scope     Governance record    Configuration #     Field Type Default Description     fields liststring  Source fields to read from.   keep bool  Invert into prune semantics: keep only the listed fields.    Example #  processor: - drop_fields: fields: [&#34;message&#34;, &#34;tags&#34;] keep: true "
---


# drop_fields

| Category | Scope |
|----------|-------|
| Governance | record |

## Configuration

| Field | Type | Default | Description |
|-------|------|---------|-------------|
| `fields` | liststring |  | Source fields to read from. |
| `keep` | bool |  | Invert into prune semantics: keep only the listed fields. |

## Example

```yaml
processor:
  - drop_fields:
      fields: ["message", "tags"]
      keep: true
```

