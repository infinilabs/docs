---
title: "RSS"
date: 0001-01-01
summary: "Rss Connector #  Register RSS Connector #  curl -XPOST &#34;http://localhost:9000/connector/&#34; -d &#39;{ &#34;name&#34; : &#34;RSS Connector&#34;, &#34;description&#34; : &#34;Fetch items from a specified RSS feed.&#34;, &#34;category&#34; : &#34;website&#34;, &#34;icon&#34; : &#34;/assets/icons/connector/rss/icon.png&#34;, &#34;tags&#34; : [ &#34;rss&#34;, &#34;feed&#34;, &#34;web&#34; ], &#34;url&#34; : &#34;http://coco.rs/connectors/rss&#34;, &#34;assets&#34; : { &#34;icons&#34; : { &#34;default&#34; : &#34;/assets/icons/connector/rss/icon.png&#34; } }, &#34;processor&#34;:{ &#34;enabled&#34;:true, &#34;name&#34;:&#34;rss&#34; } }&#39; Use the RSS Connector #  //request curl -H &#39;Content-Type: application/json&#39; -XPOST &#34;http://localhost:9000/datasource/&#34; -d&#39; { &#34;name&#34;:&#34;My RSS feed&#34;, &#34;type&#34;:&#34;connector&#34;, &#34;connector&#34;:{ &#34;id&#34;:&#34;rss&#39;s connector id&#34;, &#34;config&#34;:{ &#34;urls&#34;: [ &#34;The RSS link&#34; ] } } }&#39; Below is the config parameters supported by this connector."
---


# Rss Connector

## Register RSS Connector

```shell
curl -XPOST "http://localhost:9000/connector/" -d '{
  "name" : "RSS Connector",
  "description" : "Fetch items from a specified RSS feed.",
  "category" : "website",
  "icon" : "/assets/icons/connector/rss/icon.png",
  "tags" : [
    "rss",
    "feed",
    "web"
  ],
  "url" : "http://coco.rs/connectors/rss",
  "assets" : {
    "icons" : {
      "default" : "/assets/icons/connector/rss/icon.png"
    }
  },
     "processor":{
        "enabled":true,
        "name":"rss"
     }
}'
```

## Use the RSS Connector

```shell
//request
curl  -H 'Content-Type: application/json'   -XPOST "http://localhost:9000/datasource/" -d'
{
    "name":"My RSS feed",
    "type":"connector",
    "connector":{
        "id":"rss's connector id",
         "config":{
            "urls": [ "The RSS link" ]
        }
    }
}'
```

Below is the config parameters supported by this connector.

| **Field**              | **Type**           | **Description**                                            |
|-------------------------|--------------------|------------------------------------------------------------|
| `urls`               |  []string          | The array list of the rss feet, support more than one url. |

