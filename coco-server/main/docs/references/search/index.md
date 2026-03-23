---
title: "Search"
date: 0001-01-01
summary: "Search API #  Search API Reference #  Get Query Suggestions #  //request curl -XGET http://localhost:9000/query/_suggest\?query\=buss //response { &#34;query&#34;: &#34;buss&#34;, &#34;suggestions&#34;: [ { &#34;suggestion&#34;: &#34;Q3 Business Report&#34;, &#34;score&#34;: 0.99, &#34;source&#34;: { &#34;type&#34;:&#34;connector&#34;, &#34;name&#34;:&#34;google_drive&#34;, &#34;id&#34;:&#34;e806831dacc3&#34;, } } ] } Get Search Results #  //request curl -XGET http://localhost:9000/query/_search\?query\=Business //response {&#34;took&#34;:15,&#34;timed_out&#34;:false,&#34;_shards&#34;:{&#34;total&#34;:1,&#34;successful&#34;:1,&#34;skipped&#34;:0,&#34;failed&#34;:0},&#34;hits&#34;:{&#34;total&#34;:{&#34;value&#34;:1,&#34;relation&#34;:&#34;eq&#34;},&#34;max_score&#34;:3.0187376,&#34;hits&#34;:[{&#34;_index&#34;:&#34;coco_document&#34;,&#34;_type&#34;:&#34;_doc&#34;,&#34;_id&#34;:&#34;csstf6rq50k5sqipjaa0&#34;,&#34;_score&#34;:3.0187376,&#34;_source&#34;:{&#34;id&#34;:&#34;csstf6rq50k5sqipjaa0&#34;, ...OMITTED...}}}]}} Parameters #     Parameter Type Default Description     query string &quot;&quot; The search query string."
---


# Search API


## Search API Reference


### Get Query Suggestions

```shell
//request
curl  -XGET http://localhost:9000/query/_suggest\?query\=buss

//response
{
  "query": "buss",
  "suggestions": [
    {
      "suggestion": "Q3 Business Report",
      "score": 0.99,
      "source": {
         "type":"connector",
         "name":"google_drive",
         "id":"e806831dacc3",
       }
    }
  ]
}
```

### Get Search Results

```shell
//request
curl  -XGET http://localhost:9000/query/_search\?query\=Business

//response
{"took":15,"timed_out":false,"_shards":{"total":1,"successful":1,"skipped":0,"failed":0},"hits":{"total":{"value":1,"relation":"eq"},"max_score":3.0187376,"hits":[{"_index":"coco_document","_type":"_doc","_id":"csstf6rq50k5sqipjaa0","_score":3.0187376,"_source":{"id":"csstf6rq50k5sqipjaa0", ...OMITTED...}}}]}}
```

### Parameters

| Parameter       | Type   | Default   | Description                                                                                   |
|-----------------|--------|-----------|-----------------------------------------------------------------------------------------------|
| `query`         | string | `""`      | The search query string.                                                                      |
| `from`          | int    | `0`       | The starting offset of the search results (used for pagination).                              |
| `size`          | int    | `10`      | The number of search results to return.                                                       |
