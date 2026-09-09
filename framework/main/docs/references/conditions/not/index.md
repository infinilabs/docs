---
title: "not"
date: 0001-01-01
summary: "not #     Kind Accepts     Logical operator exactly one nested condition    Negates a single inner condition. Evaluates to true when the inner condition is false.
Parameters #     Parameter Type Description     (nested) condition The single condition to negate.    Examples #  not: contains: _ctx.request.uri: &#34;/health&#34; Notes #  Inside a per-record sub-chain (e."
---


# not

| Kind | Accepts |
|------|---------|
| Logical operator | exactly one nested condition |

Negates a single inner condition. Evaluates to `true` when the inner condition is `false`.

## Parameters

| Parameter | Type | Description |
|-----------|------|-------------|
| (nested) | condition | The single condition to negate. |

## Examples

```yaml
not:
  contains:
    _ctx.request.uri: "/health"
```

## Notes

Inside a per-record sub-chain (e.g. `for_each`), field names resolve against the current
record's own attributes (`file.path`, `log_level`, ...); at pipeline level they resolve against
the pipeline context through the `_ctx.` prefix. See the Conditions reference for details.

