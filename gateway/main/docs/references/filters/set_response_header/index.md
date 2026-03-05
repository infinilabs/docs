---
title: "set_response_header"
date: 0001-01-01
summary: "set_response_header #  Description #  The set_response_header filter is used to set the header information used in responses.
Configuration Example #  A simple example is as follows:
flow: - name: set_response_header filter: - set_response_header: headers: - Trial -&gt; true - Department -&gt; Engineering Parameter Description #     Name Type Description     headers map It uses -&gt; to identify a key value pair and set header information."
---


# set_response_header

## Description

The set_response_header filter is used to set the header information used in responses.

## Configuration Example

A simple example is as follows:

```
flow:
  - name: set_response_header
    filter:
      - set_response_header:
          headers:
            - Trial -> true
            - Department -> Engineering
```

## Parameter Description

| Name    | Type | Description                                                           |
| ------- | ---- | --------------------------------------------------------------------- |
| headers | map  | It uses `->` to identify a key value pair and set header information. |

