---
title: "replay"
date: 0001-01-01
summary: "replay #     Category Scope     General pipeline    Configuration #     Field Type Default Description     schema string &ldquo;http&rdquo; Schema of the parsed input.   host string &ldquo;localhost:9200&rdquo; Target host, e.g. http://example.com:9200.   filename string  File the resource is loaded from.   input_queue string  Queue the batches are pulled from.   username string  Basic-auth username."
---


# replay

| Category | Scope |
|----------|-------|
| General | pipeline |

## Configuration

| Field | Type | Default | Description |
|-------|------|---------|-------------|
| `schema` | string | "http" | Schema of the parsed input. |
| `host` | string | "localhost:9200" | Target host, e.g. `http://example.com:9200`. |
| `filename` | string |  | File the resource is loaded from. |
| `input_queue` | string |  | Queue the batches are pulled from. |
| `username` | string |  | Basic-auth username. |
| `password` | string |  | Basic-auth password. |

## Example

```yaml
processor:
  - replay:
      schema: "schema"
      host: "host"
      filename: "filename"
      input_queue: "input_queue"
      username: "username"
```

