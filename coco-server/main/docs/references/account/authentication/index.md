---
title: "Authentication"
date: 0001-01-01
summary: "Authentication #  Authentication Methods #  The API supports two methods of authentication:
1. Login API #  Use the X-API-TOKEN header with your token value.
Example request:
curl -XPOST http://localhost:9000/account/login -d'{ &quot;password&quot;:&quot;mypassword&quot; }' The response should be looks like this:
{ &quot;access_token&quot;: &quot;eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJleHAiOjE3NDA4Mjg5OTksInByb3ZpZGVyIjoic2ltcGxlIiwibG9naW4iOiJjb2NvLWRlZmF1bHQtdXNlciIsInVzZXJfaWQiOiJjb2NvLWRlZmF1bHQtdXNlciIsInJvbGVzIjpbXX0.iqn2uuyX7jE3H4earkW-0hbM2lK6q9Oy5lPUv0pVtLI&quot;, &quot;expire_in&quot;: 86400, &quot;id&quot;: &quot;coco-default-user&quot;, &quot;status&quot;: &quot;ok&quot;, &quot;username&quot;: &quot;coco-default-user&quot; } The access_token can be used in Bearer Authorization.
2. Bearer Authentication #  Use Basic Authentication by passing a Authorization header with the access_token returned by login API."
---


# Authentication

## Authentication Methods

The API supports two methods of authentication:

### 1. Login API

Use the X-API-TOKEN header with your token value.

Example request:
```
curl -XPOST http://localhost:9000/account/login -d'{
	"password":"mypassword"
}'
```

The response should be looks like this:
```
{
  "access_token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJleHAiOjE3NDA4Mjg5OTksInByb3ZpZGVyIjoic2ltcGxlIiwibG9naW4iOiJjb2NvLWRlZmF1bHQtdXNlciIsInVzZXJfaWQiOiJjb2NvLWRlZmF1bHQtdXNlciIsInJvbGVzIjpbXX0.iqn2uuyX7jE3H4earkW-0hbM2lK6q9Oy5lPUv0pVtLI",
  "expire_in": 86400,
  "id": "coco-default-user",
  "status": "ok",
  "username": "coco-default-user"
}
```
The `access_token` can be used in `Bearer Authorization`.

### 2. Bearer Authentication

Use Basic Authentication by passing a `Authorization` header with the `access_token` returned by login API.

Example request:

```bash
curl -XGET http://localhost:9000/<api_need_authentication> \
  -H "Authorization: Bearer <access_token>"
```

The actual example should be looks like this:
```
curl    -H "Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJleHAiOjE3NDA4Mjg5OTksInByb3ZpZGVyIjoic2ltcGxlIiwibG9naW4iOiJjb2NvLWRlZmF1bHQtdXNlciIsInVzZXJfaWQiOiJjb2NvLWRlZmF1bHQtdXNlciIsInJvbGVzIjpbXX0.iqn2uuyX7jE3H4earkW-0hbM2lK6q9Oy5lPUv0pVtLI"  http://localhost:9000/account/profile
```

### 3. API Token Authentication

Use the X-API-TOKEN header with your token value, how to get the `X-API-TOKEN` can be found in this doc: [Request API Token](./access_token.md)

Example request:
```
curl -XGET http://localhost:9000/account/profile \
  -H "X-API-TOKEN: xxxxx"
```


