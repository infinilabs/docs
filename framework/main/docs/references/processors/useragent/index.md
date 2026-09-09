---
title: "useragent"
date: 0001-01-01
summary: "useragent #     Category Scope     Parsing record    The &ldquo;useragent&rdquo; pipeline processor: a dependency-free User-Agent classifier (browser, version, os, device class) covering the common web traffic vocabulary. Output lands under.
Configuration #     Field Type Default Description     field string &ldquo;user_agent&rdquo; Source field to read from.   target_field string &ldquo;user_agent&rdquo; Destination field to write the result to."
---


# useragent

| Category | Scope |
|----------|-------|
| Parsing | record |

The "useragent" pipeline processor: a dependency-free User-Agent classifier (browser, version, os, device class) covering the common web traffic vocabulary. Output lands under.

## Configuration

| Field | Type | Default | Description |
|-------|------|---------|-------------|
| `field` | string | "user_agent" | Source field to read from. |
| `target_field` | string | "user_agent" | Destination field to write the result to. |
| `ignore_missing` | bool |  | Do not fail when the source field is missing. |

## Example

```yaml
processor:
  - useragent:
      field: "message"
      target_field: "parsed"
      ignore_missing: true
```

