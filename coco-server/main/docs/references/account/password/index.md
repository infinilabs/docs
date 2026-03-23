---
title: "Modify Password"
date: 0001-01-01
summary: "Modify Password #  Modify the current user&rsquo;s password.
curl -XPUT http://localhost:9000/account/password -d'{ &quot;old_password&quot;:&quot;xxxx&quot;, &quot;new_password&quot;:&quot;xxxx&quot; }' "
---


# Modify Password

Modify the current user's password.

```
curl -XPUT http://localhost:9000/account/password -d'{
	"old_password":"xxxx",
	"new_password":"xxxx"
}'
```
