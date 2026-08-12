---
title: "basic_auth"
date: 0001-01-01
summary: "basic_auth #  Description #  The basic_auth filter is used to verify authentication information of requests. It is applicable to simple authentication.
Configuration Example #  A simple example is as follows:
flow: - name: basic_auth filter: - basic_auth: valid_users: medcl: passwd medcl1: abc ... Parameter Description #     Name Type Description     valid_users map Username and password    "
---


# basic_auth

## Description

The basic_auth filter is used to verify authentication information of requests. It is applicable to simple authentication.

## Configuration Example

A simple example is as follows:

```
flow:
  - name: basic_auth
    filter:
      - basic_auth:
          valid_users:
            medcl: passwd
            medcl1: abc
            ...
```

## Parameter Description

| Name        | Type | Description           |
| ----------- | ---- | --------------------- |
| valid_users | map  | Username and password |

