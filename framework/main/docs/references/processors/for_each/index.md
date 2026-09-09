---
title: "for_each"
date: 0001-01-01
summary: "for_each #     Category Scope     Framework batch    Configuration #     Field Type Default Description     message_field string &ldquo;messages&rdquo; Context key holding the message batch.   codec string  Payload codec used to decode/encode records (default otel).   on_failure string  Sub-chain failure strategy: ignore, tag or fail.   failure_tag string &ldquo;_processing_failed&rdquo; Tag appended to the record when on_failure is tag."
---


# for_each

| Category | Scope |
|----------|-------|
| Framework | batch |

## Configuration

| Field | Type | Default | Description |
|-------|------|---------|-------------|
| `message_field` | string | "messages" | Context key holding the message batch. |
| `codec` | string |  | Payload codec used to decode/encode records (default `otel`). |
| `on_failure` | string |  | Sub-chain failure strategy: `ignore`, `tag` or `fail`. |
| `failure_tag` | string | "_processing_failed" | Tag appended to the record when `on_failure` is `tag`. |
| `processor` | list (processor configs) |  | Ordered sub-chain of processors executed per record. |

## Example

```yaml
processor:
  - for_each:
      codec: otel
      on_failure: tag
      processor:
        - dissect:
            pattern: "%{log_level} %{message}"
        - if:
            equals:
              log_level: error
          then:
            - drop_event:
```

