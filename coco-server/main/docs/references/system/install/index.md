---
title: "System Initialization"
date: 0001-01-01
summary: "System Initialization #  Initialization API #  curl -XPOST http://localhost:9000/setup/_initialize -d' { &quot;name&quot;:&quot;Coco&quot;, &quot;email&quot;:&quot;hello@coco.rs&quot;, &quot;password&quot;:&quot;mypassword&quot;, &quot;llm&quot;:{ &quot;type&quot;:&quot;ollama&quot;, &quot;endpoint&quot;:&quot;http://xxx&quot;, &quot;default_model&quot;:&quot;deepseek_r1&quot; } }' Initialization UI Management #  When entering the coco server, it will check whether it has been initialized. If not, it will enter the initialization page.
Create a user account #  Set up a new user account to manage access and permissions.
Connect to a Large Model #  After integrating a large model, you will unlock the AI chat feature, providing intelligent search and an efficient work assistant."
---


# System Initialization

## Initialization API
```
curl -XPOST http://localhost:9000/setup/_initialize -d'
{
	"name":"Coco",
	"email":"hello@coco.rs",
	"password":"mypassword",
	"llm":{
		  "type":"ollama",
		  "endpoint":"http://xxx",
		  "default_model":"deepseek_r1"
	}
}'
```

## Initialization UI Management

When entering the coco server, it will check whether it has been initialized. If not, it will enter the initialization page.

### Create a user account

Set up a new user account to manage access and permissions.

{{% load-img "/img/initialization/step-1.png" "initialization step 1" %}}

### Connect to a Large Model

After integrating a large model, you will unlock the AI chat feature, providing intelligent search and an efficient work assistant.

{{% load-img "/img/initialization/step-2.png" "initialization step 2" %}}

You can also click `Set Up Later` to skip and configure it later after entering the system.


