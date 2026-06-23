---
title: "Logout"
date: 0001-01-01
summary: "Logout #  Logout API #  The Logout API securely logs the user out from the Coco Server. It destroys the current session.
Both GET and POST methods are supported.
//request curl -XPOST http://localhost:9000/account/logout \  -H &#34;Authorization: Bearer &lt;access_token&gt;&#34; //response { &#34;status&#34;: &#34;ok&#34; } "
---


# Logout

## Logout API

The Logout API securely logs the user out from the Coco Server. It destroys the current session.

Both `GET` and `POST` methods are supported.

```shell
//request
curl -XPOST http://localhost:9000/account/logout \
  -H "Authorization: Bearer <access_token>"

//response
{
  "status": "ok"
}
```
