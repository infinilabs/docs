---
title: "set_request_header"
date: 0001-01-01
summary: "set_request_header #  Description #  The set_request_header filter is used to set header information for requests.
Configuration Example #  A simple example is as follows:
flow: - name: set_request_header filter: - set_request_header: headers: - Trial -&gt; true - Department -&gt; Engineering Parameter Description #     Name Type Description     headers map It uses -&gt; to identify a key value pair and set header information."
---


# set_request_header

## Description

The set_request_header filter is used to set header information for requests.

## Configuration Example

A simple example is as follows:

```
flow:
  - name: set_request_header
    filter:
      - set_request_header:
          headers:
            - Trial -> true
            - Department -> Engineering
```

## Parameter Description

| Name    | Type | Description                                                           |
| ------- | ---- | --------------------------------------------------------------------- |
| headers | map  | It uses `->` to identify a key value pair and set header information. |

