---
title: "add_locale"
date: 0001-01-01
summary: "add_locale #     Category Scope     Enrichment record    The &ldquo;add_locale&rdquo; pipeline processor: stamp each event with the host&rsquo;s local timezone abbreviation or UTC offset, so downstream parsing can interpret naive timestamps correctly.
Configuration #     Field Type Default Description     format string &ldquo;offset&rdquo; Preset format selector (e.g. log format).   target string &ldquo;event.timezone&rdquo; Destination prefix or field to write the result to."
---


# add_locale

| Category | Scope |
|----------|-------|
| Enrichment | record |

The "add_locale" pipeline processor: stamp each event with the host's local timezone abbreviation or UTC offset, so downstream parsing can interpret naive timestamps correctly.

## Configuration

| Field | Type | Default | Description |
|-------|------|---------|-------------|
| `format` | string | "offset" | Preset format selector (e.g. log format). |
| `target` | string | "event.timezone" | Destination prefix or field to write the result to. |

## Example

```yaml
processor:
  - add_locale:
      format: "format"
      target: "parsed"
```

