---
title: "registered_domain"
date: 0001-01-01
summary: "registered_domain #     Category Scope     Enrichment record    The &ldquo;registered_domain&rdquo; pipeline processor: split a domain (e.g. url.domain / dns.question.name) into its registered domain (&ldquo;www.example.co.uk&rdquo; -&gt; &ldquo;example.co.uk&rdquo;) using the Mozilla Public Suffix List (golang.org/x/net/publicsuffix).
Configuration #     Field Type Default Description     field string  Source field to read from.   target_field string &ldquo;network.registered_domain&rdquo; Destination field to write the result to."
---


# registered_domain

| Category | Scope |
|----------|-------|
| Enrichment | record |

The "registered_domain" pipeline processor: split a domain (e.g. url.domain / dns.question.name) into its registered domain ("www.example.co.uk" -> "example.co.uk") using the Mozilla Public Suffix List (golang.org/x/net/publicsuffix).

## Configuration

| Field | Type | Default | Description |
|-------|------|---------|-------------|
| `field` | string |  | Source field to read from. |
| `target_field` | string | "network.registered_domain" | Destination field to write the result to. |
| `target_subdomain_field` | string |  | Field receiving the subdomain part. |
| `target_etld_field` | string |  | Field receiving the extracted registrable domain. |
| `ignore_missing` | bool | true | Do not fail when the source field is missing. |
| `ignore_failure` | bool |  | Do not fail the record when processing errors; the record passes through unchanged. |

## Example

```yaml
processor:
  - registered_domain:
      field: "message"
      target_field: "parsed"
      target_subdomain_field: "target_subdomain_field"
      target_etld_field: "target_etld_field"
      ignore_missing: true
```

