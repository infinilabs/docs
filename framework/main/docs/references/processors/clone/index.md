---
title: "clone"
date: 0001-01-01
summary: "clone #     Category Scope     Routing record    The &ldquo;clone&rdquo; pipeline processor: duplicate the current record N times (Graylog&rsquo;s clone_message). Clones carry the same base fields plus an optional mutations map per clone, so one pass can fan a record out into several variants for per-variant downstream processing (e.g. one clone tagged for metrics, one for archival).
Configuration #     Field Type Default Description     count int 1 Number of copies produced."
---


# clone

| Category | Scope |
|----------|-------|
| Routing | record |

The "clone" pipeline processor: duplicate the current record N times (Graylog's clone_message). Clones carry the same base fields plus an optional mutations map per clone, so one pass can fan a record out into several variants for per-variant downstream processing (e.g. one clone tagged for metrics, one for archival).

## Configuration

| Field | Type | Default | Description |
|-------|------|---------|-------------|
| `count` | int | 1 | Number of copies produced. |
| `mutate` | listmap |  | per-clone field overrides |

## Example

```yaml
processor:
  - clone:
      count: 10
      mutate: []
```

