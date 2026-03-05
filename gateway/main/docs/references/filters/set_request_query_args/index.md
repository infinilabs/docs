---
title: "set_request_query_args"
date: 0001-01-01
summary: "set_request_query_args #  Description #  The set_request_query_args filter is used to set the QueryString parameter information used for requests.
Configuration Example #  A simple example is as follows:
flow: - name: set_request_query_args filter: - set_request_query_args: args: - size -&gt; 10 Parameter Description #     Name Type Description     args map It uses -&gt; to identify a key value pair and set QueryString parameter information."
---


# set_request_query_args

## Description

The set_request_query_args filter is used to set the QueryString parameter information used for requests.

## Configuration Example

A simple example is as follows:

```
flow:
  - name: set_request_query_args
    filter:
      - set_request_query_args:
          args:
            - size -> 10
```

## Parameter Description

| Name | Type | Description                                                                          |
| ---- | ---- | ------------------------------------------------------------------------------------ |
| args | map  | It uses `->` to identify a key value pair and set QueryString parameter information. |

