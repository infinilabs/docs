---
title: "Local FS"
date: 0001-01-01
summary: "Local FS Connector #  Register Local FS Connector #  curl -XPOST &#34;http://localhost:9000/connector/&#34; -d &#39;{ &#34;name&#34; : &#34;Local Filesystem Connector&#34;, &#34;description&#34; : &#34;Scan and fetch metadata from local files.&#34;, &#34;category&#34; : &#34;local_storage&#34;, &#34;icon&#34; : &#34;/assets/icons/connector/local_fs/icon.png&#34;, &#34;tags&#34; : [ &#34;storage&#34;, &#34;filesystem&#34; ], &#34;url&#34; : &#34;http://coco.rs/connectors/local_fs&#34;, &#34;assets&#34; : { &#34;icons&#34; : { &#34;default&#34; : &#34;/assets/icons/connector/local_fs/icon.png&#34; } } }&#39; Use the Local FS Connector #  The Local FS Connector allows you to index data from your local filesystem into your system."
---


# Local FS Connector

## Register Local FS Connector

```shell
curl -XPOST "http://localhost:9000/connector/" -d '{
  "name" : "Local Filesystem Connector",
  "description" : "Scan and fetch metadata from local files.",
  "category" : "local_storage",
  "icon" : "/assets/icons/connector/local_fs/icon.png",
  "tags" : [
    "storage",
    "filesystem"
  ],
  "url" : "http://coco.rs/connectors/local_fs",
  "assets" : {
    "icons" : {
      "default" : "/assets/icons/connector/local_fs/icon.png"
    }
  }
}'
```

## Use the Local FS Connector

The Local FS Connector allows you to index data from your local filesystem into your system. Follow these steps to set it up:

### Configure file folders & extensions

You need to configure the folder path, and the connector will scan the metadata of all files under the folder, including subfolders.
You can add file extension configuration, and the connector will only scan files with the specified extension you specify.

### Example Request

Here is an example request to configure the Notion Connector:

```shell
curl -H 'Content-Type: application/json' -XPOST "http://localhost:9000/datasource/" -d '
{
    "name":"My Local Documents",
    "type":"connector",
    "connector":{
        "id":"local_fs",
         "config":{
            "paths": [ "/path/to/my/documents", "/path/to/another/folder" ],
            "extensions": [ "pdf", "docx", "txt" ]
        }
    }
}''
```

## Supported Config Parameters for Local FS Connector

Below are the configuration parameters supported by the Local FS Connector:

| **Field**               | **Type**    | **Description**                                                                                                     |
|--------------------------|-------------|---------------------------------------------------------------------------------------------------------------------|
| `paths`   | `[]string`  | An array of absolute paths to the folders you want to scan.                                                         |
| `extensions`  | `[]string`  | Optional. An array of file extensions to include (e.g., pdf, docx). If omitted or empty, all files will be indexed. |


