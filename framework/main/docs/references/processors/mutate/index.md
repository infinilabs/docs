---
title: "mutate"
date: 0001-01-01
summary: "mutate #     Category Scope     Transform record    The &ldquo;mutate&rdquo; pipeline processor: field-level mutations on the current record, covering the Logstash mutate filter surface. Actions apply in a fixed order: rename, copy, add, convert, replace, lowercase, uppercase, strip, remove.
Configuration #     Field Type Default Description     pattern string  Extraction pattern; %{NAME} placeholders capture into fields."
---


# mutate

| Category | Scope |
|----------|-------|
| Transform | record |

The "mutate" pipeline processor: field-level mutations on the current record, covering the Logstash mutate filter surface. Actions apply in a fixed order: rename, copy, add, convert, replace, lowercase, uppercase, strip, remove.

## Configuration

| Field | Type | Default | Description |
|-------|------|---------|-------------|
| `pattern` | string |  | Extraction pattern; `%{NAME}` placeholders capture into fields. |
| `to` | string |  | Destination of the rendered value. |

## Example

```yaml
processor:
  - for_each:
      processor:
        - mutate:
            rename:
              lvl: log_level
            add:
              env: prod
            convert:
              status_code: integer
            lowercase:
              - log_level
            remove:
              - tmp_field
```

