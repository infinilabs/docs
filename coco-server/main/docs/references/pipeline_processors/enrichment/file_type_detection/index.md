---
title: "File Type Detection"
date: 0001-01-01
summary: "File Type Detection Processor #  Detects file types based on file extensions and sets appropriate metadata.
Configuration #     Parameter Type Default Description     message_field string documents The field in the pipeline context containing the documents to process   output_queue object null Optional queue configuration for sending processed documents to a output queue    Example #  - file_type_detection: {} "
---



## File Type Detection Processor

Detects file types based on file extensions and sets appropriate metadata.

### Configuration

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| `message_field` | string | `documents` | The field in the pipeline context containing the documents to process |
| `output_queue` | object | `null` | Optional queue configuration for sending processed documents to a output queue |

### Example

```yaml
- file_type_detection: {}
```
