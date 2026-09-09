---
title: "queue_has_lag"
date: 0001-01-01
summary: "queue_has_lag #     Kind Accepts     Domain condition a list of queue specifiers    Tests whether a message queue has unconsumed messages. An optional &gt; max_depth threshold can be appended to a queue specifier.
Parameters #     Parameter Type Description     (list) list of strings Queue names, optionally &quot;queue &gt; threshold&quot;.    Examples #  queue_has_lag: - &#34;my_queue&#34; - &#34;my_queue &gt; 1000&#34; Notes #  Inside a per-record sub-chain (e."
---


# queue_has_lag

| Kind | Accepts |
|------|---------|
| Domain condition | a list of queue specifiers |

Tests whether a message queue has unconsumed messages. An optional `> max_depth` threshold can be appended to a queue specifier.

## Parameters

| Parameter | Type | Description |
|-----------|------|-------------|
| (list) | list of strings | Queue names, optionally `"queue > threshold"`. |

## Examples

```yaml
queue_has_lag:
  - "my_queue"
  - "my_queue > 1000"
```

## Notes

Inside a per-record sub-chain (e.g. `for_each`), field names resolve against the current
record's own attributes (`file.path`, `log_level`, ...); at pipeline level they resolve against
the pipeline context through the `_ctx.` prefix. See the Conditions reference for details.

