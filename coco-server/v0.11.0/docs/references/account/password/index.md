---
title: "Modify Password"
date: 0001-01-01
summary: "Modify Password #  Modify the current user&rsquo;s password. Requires authentication.
Modify Password #  //request curl -XPUT http://localhost:9000/account/password \  -H &#34;Authorization: Bearer &lt;access_token&gt;&#34; \  -H &#39;Content-Type: application/json&#39; \  -d&#39;{ &#34;old_password&#34;:&#34;current_password&#34;, &#34;new_password&#34;:&#34;new_secure_password&#34; }&#39; //response { &#34;_id&#34;: &#34;coco-default-user&#34;, &#34;result&#34;: &#34;updated&#34; } Parameters #     Field Type Required Description     old_password string Yes The user&rsquo;s current password.   new_password string Yes The new password to set."
---


# Modify Password

Modify the current user's password. Requires authentication.

## Modify Password

```shell
//request
curl -XPUT http://localhost:9000/account/password \
  -H "Authorization: Bearer <access_token>" \
  -H 'Content-Type: application/json' \
  -d'{
  "old_password":"current_password",
  "new_password":"new_secure_password"
}'

//response
{
  "_id": "coco-default-user",
  "result": "updated"
}
```

### Parameters

| **Field**      | **Type** | **Required** | **Description**                   |
|----------------|----------|--------------|-----------------------------------|
| `old_password` | `string` | Yes          | The user's current password.      |
| `new_password` | `string` | Yes          | The new password to set.          |
