---
title: "set_hostname"
date: 0001-01-01
summary: "set_hostname #  Description #  The set_hostname filter is used to set the host or domain name to be accessed in the request header.
Configuration Example #  A simple example is as follows:
flow: - name: set_hostname filter: - set_hostname: hostname: api.infini.cloud Parameter Description #     Name Type Description     hostname string Host information    "
---


# set_hostname

## Description

The set_hostname filter is used to set the host or domain name to be accessed in the request header.

## Configuration Example

A simple example is as follows:

```
flow:
  - name: set_hostname
    filter:
      - set_hostname:
          hostname: api.infini.cloud
```

## Parameter Description

| Name     | Type   | Description      |
| -------- | ------ | ---------------- |
| hostname | string | Host information |

