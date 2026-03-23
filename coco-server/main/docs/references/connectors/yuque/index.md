---
title: "Yuque"
date: 0001-01-01
summary: "Yuque Connector #  Register Yuque Connector #  curl -XPOST &#34;http://localhost:9000/connector&#34; -d &#39;{ &#34;name&#34;: &#34;Yuque Docs Connector&#34;, &#34;description&#34;: &#34;Fetch the docs metadata for yuque.&#34;, &#34;icon&#34;: &#34;/assets/connector/yuque/icon.png&#34;, &#34;category&#34;: &#34;website&#34;, &#34;tags&#34;: [ &#34;static_site&#34;, &#34;hugo&#34;, &#34;web&#34; ], &#34;url&#34;: &#34;http://coco.rs/connectors/hugo_site&#34;, &#34;assets&#34;: { &#34;icons&#34;: { &#34;default&#34;: &#34;/assets/connector/yuque/icon.png&#34;, &#34;book&#34;: &#34;/assets/connector/yuque/book.png&#34;, &#34;board&#34;: &#34;/assets/connector/yuque/board.png&#34;, &#34;sheet&#34;: &#34;/assets/connector/yuque/sheet.png&#34;, &#34;table&#34;: &#34;/assets/connector/yuque/table.png&#34;, &#34;doc&#34;: &#34;/assets/connector/yuque/doc.png&#34; } }, &#34;processor&#34;:{ &#34;enabled&#34;:true, &#34;name&#34;:&#34;yuque&#34; } }&#39; Use the Yuque Connector #  The Yuque Connector allows you to index data from your Yuque account into your system."
---


# Yuque Connector

## Register Yuque Connector

```shell
curl -XPOST "http://localhost:9000/connector" -d '{
    "name": "Yuque Docs Connector", 
    "description": "Fetch the docs metadata for yuque.", 
    "icon": "/assets/connector/yuque/icon.png", 
    "category": "website", 
    "tags": [
        "static_site", 
        "hugo", 
        "web"
    ], 
    "url": "http://coco.rs/connectors/hugo_site", 
    "assets": {
        "icons": {
            "default": "/assets/connector/yuque/icon.png", 
            "book": "/assets/connector/yuque/book.png", 
            "board": "/assets/connector/yuque/board.png", 
            "sheet": "/assets/connector/yuque/sheet.png", 
            "table": "/assets/connector/yuque/table.png", 
            "doc": "/assets/connector/yuque/doc.png"
        }
    },
       "processor":{
          "enabled":true,
          "name":"yuque"
       }
}'
```

## Use the Yuque Connector

The Yuque Connector allows you to index data from your Yuque account into your system. Follow these steps to set it up:

### Obtain Yuque API Token

Before using this connector, you need to obtain your Yuque API token. Refer to the official [Yuque API documentation](https://www.yuque.com/yuque/developer/api) for instructions.

### Example Request

Here is an example request to configure the Yuque Connector:

```shell
curl -H 'Content-Type: application/json' -XPOST "http://localhost:9000/datasource/" -d '
{
    "name": "My Yuque",
    "type": "connector",
    "connector": {
        "id": "yuque's connector id",
        "config": {
            "token": "your_yuque_api_token",
            "include_private_book": false,
            "include_private_doc": false,
            "indexing_books": true,
            "indexing_docs": true,
            "indexing_users": true,
            "indexing_groups": true
        }
    }
}'
```

## Supported Config Parameters for Yuque Connector

Below are the configuration parameters supported by the Yuque Connector:

| **Field**               | **Type**  | **Description**                                                                                  |
|--------------------------|-----------|--------------------------------------------------------------------------------------------------|
| `token`                 | `string`  | Your Yuque API token. This is required to access Yuque's API.                                    |
| `include_private_book`  | `bool`    | Whether to include private books in indexing. Defaults to `false`.                              |
| `include_private_doc`   | `bool`    | Whether to include private documents in indexing. Defaults to `false`.                          |
| `indexing_books`        | `bool`    | Whether to index books in Yuque. Defaults to `false`.                                           |
| `indexing_docs`         | `bool`    | Whether to index documents in Yuque. Defaults to `false`.                                       |
| `indexing_users`        | `bool`    | Whether to index user data from Yuque. Defaults to `false`.                                     |
| `indexing_groups`       | `bool`    | Whether to index group data from Yuque. Defaults to `false`.                                    |

### Notes
- Update `yuque's connector id` to your actual connector ID
- Set `token` to your valid Yuque API token to enable the connector.
- Boolean parameters like `include_private_book`, `indexing_books`, etc., allow you to customize the scope of indexing.
