---
title: "System Settings"
date: 0001-01-01
summary: "System Settings #  System Settings API #  curl -XPOST http://localhost:9000/settings -d' { &quot;server&quot;:{ &quot;name&quot;: &quot;My Coco Server&quot;, &quot;endpoint&quot;:&quot;http://xxxx/&quot;, &quot;provider&quot;: { &quot;banner&quot;: &quot;http://localhost:9000/banner2.jpg&quot;, &quot;description&quot;: &quot;Coco AI Server - Search, Connect, Collaborate, AI-powered enterprise search, all in one space.&quot;, &quot;eula&quot;: &quot;http://infinilabs.com/eula.txt&quot;, &quot;icon&quot;: &quot;http://localhost:9000/icon.png&quot;, &quot;name&quot;: &quot;INFINI Labs&quot;, &quot;privacy_policy&quot;: &quot;http://infinilabs.com/privacy_policy.txt&quot;, &quot;website&quot;: &quot;http://infinilabs.com&quot; }, }, &quot;llm&quot;:{ &quot;type&quot;:&quot;ollama&quot;, //or openai &quot;endpoint&quot;:&quot;http://xxx&quot;, &quot;default_model&quot;:&quot;deepseek_r1&quot;, &quot;parameters&quot;:{ &quot;top_p&quot;:111, &quot;max_tokens&quot;:32000, &quot;presence_penalty&quot;:0.9, &quot;frequency_penalty&quot;:0.9, &quot;enhanced_inference&quot;:true, } } }' System Settings UI Management #  Server #  Log in to the Coco-Server admin dashboard, click Home in the left menu to view server infomation, as shown below:"
---


# System Settings

## System Settings API

```
curl -XPOST http://localhost:9000/settings -d'
{
   "server":{
   		"name": "My Coco Server",
        "endpoint":"http://xxxx/",
		"provider": {
			"banner": "http://localhost:9000/banner2.jpg",
			"description": "Coco AI Server - Search, Connect, Collaborate, AI-powered enterprise search, all in one space.",
			"eula": "http://infinilabs.com/eula.txt",
			"icon": "http://localhost:9000/icon.png",
			"name": "INFINI Labs",
			"privacy_policy": "http://infinilabs.com/privacy_policy.txt",
			"website": "http://infinilabs.com"
		},

   },
	"llm":{
		  "type":"ollama", //or openai
		  "endpoint":"http://xxx",
		  "default_model":"deepseek_r1",
		  "parameters":{
			  "top_p":111,
			  "max_tokens":32000,
			  "presence_penalty":0.9,
			  "frequency_penalty":0.9,
			  "enhanced_inference":true,
		  }
	}
}'
```

## System Settings UI Management

### Server

Log in to the Coco-Server admin dashboard, click `Home` in the left menu to view server infomation, as shown below:

{{% load-img "/img/settings/settings-server.png" "settings server" %}}

Click the edit button next to the name (address), enter a new one, and then click the save button to save the name (address).

### App Settings

Log in to the Coco-Server admin dashboard, click `Settings` in the left menu, and then click tab `App Settings` to view app settings, as shown below:

{{% load-img "/img/settings/settings-app-1.png" "settings server" %}}

You can set whether to enable the chat start page and configure the relevant parameters here.

{{% load-img "/img/settings/settings-app-2.png" "settings server" %}}




