---
title: "fingerprint"
date: 0001-01-01
summary: "fingerprint #     Category Scope     Enrichment record    Configuration #     Field Type Default Description     fields liststring  Source fields to read from.   target string  Destination prefix or field to write the result to.   method string &ldquo;sha256&rdquo; HTTP method of the request.    Example #  processor: - fingerprint: fields: [&#34;message&#34;, &#34;tags&#34;] target: &#34;parsed&#34; method: &#34;GET&#34; "
---


# fingerprint

| Category | Scope |
|----------|-------|
| Enrichment | record |

## Configuration

| Field | Type | Default | Description |
|-------|------|---------|-------------|
| `fields` | liststring |  | Source fields to read from. |
| `target` | string |  | Destination prefix or field to write the result to. |
| `method` | string | "sha256" | HTTP method of the request. |

## Example

```yaml
processor:
  - fingerprint:
      fields: ["message", "tags"]
      target: "parsed"
      method: "GET"
```

