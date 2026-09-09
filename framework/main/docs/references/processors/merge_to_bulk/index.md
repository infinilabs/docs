---
title: "merge_to_bulk"
date: 0001-01-01
summary: "merge_to_bulk #     Category Scope     Elasticsearch pipeline    Configuration #     Field Type Default Description     message_field param.ParaKey &ldquo;messages&rdquo; Context key holding the message batch.   bulk_size_in_kb int  Maximum bulk request size in kilobytes.   bulk_size_in_mb int 10 Maximum bulk request size in megabytes.   elasticsearch string  ID of the registered Elasticsearch/Easysearch cluster."
---


# merge_to_bulk

| Category | Scope |
|----------|-------|
| Elasticsearch | pipeline |

## Configuration

| Field | Type | Default | Description |
|-------|------|---------|-------------|
| `message_field` | param.ParaKey | "messages" | Context key holding the message batch. |
| `bulk_size_in_kb` | int |  | Maximum bulk request size in kilobytes. |
| `bulk_size_in_mb` | int | 10 | Maximum bulk request size in megabytes. |
| `elasticsearch` | string |  | ID of the registered Elasticsearch/Easysearch cluster. |
| `index_name` | string |  | Destination index name; supports templating. |
| `type_name` | string |  | Elasticsearch mapping type of the documents. |
| `name` | string |  | Identifier of the resource this processor binds to. |
| `label` | map |  | Static labels attached to the documents handled by this processor. |

## Example

```yaml
processor:
  - merge_to_bulk:
      message_field: "message_field"
      bulk_size_in_kb: 10
      bulk_size_in_mb: 10
      elasticsearch: "default"
      index_name: "index_name"
```

