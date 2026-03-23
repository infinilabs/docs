---
title: "Connectors"
date: 0001-01-01
summary: "Connectors #  Work with Connector #  Register a Connector #  //request curl -H &#39;Content-Type: application/json&#39; -XPOST http://localhost:9000/connector/ -d&#39;{ &#34;name&#34;: &#34;Hugo Site Connector&#34;, &#34;description&#34;: &#34;Fetch the index.json file from a specified Hugo site.&#34;, &#34;icon&#34;: &#34;http://coco.infini.cloud/assets/hugo.png&#34;, &#34;category&#34;: &#34;website&#34;, &#34;tags&#34;: [ &#34;static_site&#34;, &#34;hugo&#34;, &#34;web&#34; ], &#34;url&#34;: &#34;http://coco.rs/connectors/hugo_site&#34;, &#34;assets&#34;: { &#34;icons&#34;: { &#34;default&#34;: &#34;http://coco.infini.cloud/assets/web.png&#34;, &#34;blog&#34;: &#34;http://coco.infini.cloud/assets/hugo/blog.png&#34;, &#34;web&#34;: &#34;http://coco.infini.cloud/assets/hugo/web_page.png&#34;, &#34;news&#34;: &#34;http://coco.infini.cloud/assets/hugo/news.png&#34; } }, &#34;processor&#34;:{ &#34;enabled&#34;:true, &#34;name&#34;:&#34;hugo&#34; } }&#39; //response { &#34;_id&#34;: &#34;cu0caqt3q95r66at41o0&#34;, &#34;result&#34;: &#34;created&#34; }  Note: Every connector is expected to implement a processor that can be registered through the API."
---


# Connectors

## Work with *Connector*

### Register a Connector

```shell
//request
curl  -H 'Content-Type: application/json'   -XPOST http://localhost:9000/connector/ -d'{
    "name": "Hugo Site Connector",
    "description": "Fetch the index.json file from a specified Hugo site.",
    "icon": "http://coco.infini.cloud/assets/hugo.png",
    "category": "website",
    "tags": [
        "static_site",
        "hugo",
        "web"
    ],
    "url": "http://coco.rs/connectors/hugo_site",
    "assets": {
        "icons": {
            "default": "http://coco.infini.cloud/assets/web.png",
            "blog": "http://coco.infini.cloud/assets/hugo/blog.png",
            "web": "http://coco.infini.cloud/assets/hugo/web_page.png",
            "news": "http://coco.infini.cloud/assets/hugo/news.png"
        }
    },
    "processor":{
       "enabled":true,
       "name":"hugo"
    }
}'

//response
{
  "_id": "cu0caqt3q95r66at41o0",
  "result": "created"
}
```

> Note: Every connector is expected to implement a processor that can be registered through the API. For instance, the built-in `hugo` processor is demonstrated in the example above.

### Update a Connector
```shell
curl -XPUT http://localhost:9000/connector/cu0caqt3q95r66at41o0  -d '{
    "name": "Hugo Site Connector",
    "description": "Fetch the index.json file from a specified Hugo site.",
    "icon": "http://coco.infini.cloud/assets/hugo.png",
    "category": "website",
    "tags": [
        "static_site",
        "hugo",
        "web"
    ],
    "url": "http://coco.rs/connectors/hugo_site",
    "assets": {
        "icons": {
            "default": "http://coco.infini.cloud/assets/web.png",
            "blog": "http://coco.infini.cloud/assets/hugo/blog.png",
            "web": "http://coco.infini.cloud/assets/hugo/web_page.png",
            "news": "http://coco.infini.cloud/assets/hugo/news.png"
        }
    },
    "processor":{
       "enabled":true,
       "name":"hugo"
    }
}'

//response
{
  "_id": "cu0caqt3q95r66at41o0",
  "result": "updated"
}
```

> `?replace=true` can safely ignore errors for non-existent items.

### View a Connector
```shell
curl -XGET http://localhost:9000/connector/cu0caqt3q95r66at41o0
```

### Delete the Connector
```shell
curl -XDELETE http://localhost:9000/connector/cu0caqt3q95r66at41o0

//response
{
  "_id": "cu0caqt3q95r66at41o0",
  "result": "deleted"
}
```

### Search Connectors
```shell
curl -XGET http://localhost:9000/connector/_search
```
