---
title: "Geo Bounding Box 查询"
date: 0001-01-01
description: "矩形区域过滤：参数、坐标格式与用法。"
summary: "Geo Bounding Box 查询 #  地理边界框查询返回 geo_point 字段值位于指定矩形边界框内的文档。如果一个文档包含多个地理坐标点，只要至少有一个点位于边界框内，该文档就匹配。
参考样例 #  您可以使用地理边界框查询来搜索包含地理点的文档。
创建一个映射，将 point 字段映射为 geo_point ：
PUT testindex1 { &#34;mappings&#34;: { &#34;properties&#34;: { &#34;point&#34;: { &#34;type&#34;: &#34;geo_point&#34; } } } } 以经纬度作为对象索引三个地理点：
PUT testindex1/_doc/1 { &#34;point&#34;: { &#34;lat&#34;: 74.00, &#34;lon&#34;: 40.71 } } PUT testindex1/_doc/2 { &#34;point&#34;: { &#34;lat&#34;: 72.64, &#34;lon&#34;: 22.62 } } PUT testindex1/_doc/3 { &#34;point&#34;: { &#34;lat&#34;: 75.00, &#34;lon&#34;: 28.00 } } 搜索所有文档，并筛选出查询中定义的矩形内的点所在的文档：
GET testindex1/_search { &#34;query&#34;: { &#34;bool&#34;: { &#34;must&#34;: { &#34;match_all&#34;: {} }, &#34;filter&#34;: { &#34;geo_bounding_box&#34;: { &#34;point&#34;: { &#34;top_left&#34;: { &#34;lat&#34;: 75, &#34;lon&#34;: 28 }, &#34;bottom_right&#34;: { &#34;lat&#34;: 73, &#34;lon&#34;: 41 } } } } } } } 返回内容包含匹配的文档："
---


# Geo Bounding Box 查询

地理边界框查询返回 `geo_point` 字段值位于指定矩形边界框内的文档。如果一个文档包含多个地理坐标点，只要至少有一个点位于边界框内，该文档就匹配。

## 参考样例

您可以使用地理边界框查询来搜索包含地理点的文档。

创建一个映射，将 `point` 字段映射为 `geo_point` ：

```
PUT testindex1
{
  "mappings": {
    "properties": {
      "point": {
        "type": "geo_point"
      }
    }
  }
}
```

以经纬度作为对象索引三个地理点：

```
PUT testindex1/_doc/1
{
  "point": {
    "lat": 74.00,
    "lon": 40.71
  }
}

PUT testindex1/_doc/2
{
  "point": {
    "lat": 72.64,
    "lon": 22.62
  }
}

PUT testindex1/_doc/3
{
  "point": {
    "lat": 75.00,
    "lon": 28.00
  }
}
```

搜索所有文档，并筛选出查询中定义的矩形内的点所在的文档：

```
GET testindex1/_search
{
  "query": {
    "bool": {
      "must": {
        "match_all": {}
      },
      "filter": {
        "geo_bounding_box": {
          "point": {
            "top_left": {
              "lat": 75,
              "lon": 28
            },
            "bottom_right": {
              "lat": 73,
              "lon": 41
            }
          }
        }
      }
    }
  }
}
```

返回内容包含匹配的文档：

```
{
  "took" : 20,
  "timed_out" : false,
  "_shards" : {
    "total" : 1,
    "successful" : 1,
    "skipped" : 0,
    "failed" : 0
  },
  "hits" : {
    "total" : {
      "value" : 1,
      "relation" : "eq"
    },
    "max_score" : 1.0,
    "hits" : [
      {
        "_index" : "testindex1",
        "_id" : "1",
        "_score" : 1.0,
        "_source" : {
          "point" : {
            "lat" : 74.0,
            "lon" : 40.71
          }
        }
      }
    ]
  }
}
```

> 前一个响应未包含具有地理点 "lat": 75.00, "lon": 28.00 的文档，因为地理点的精度有限。

## 关于精度

地理点坐标在索引时总是向下取整。在查询时，边界框的上边界向下取整，下边界向上取整。因此，位于边界框下和左边缘的地理点文档可能因舍入误差而未包含在结果中。另一方面，即使它们位于边界之外，位于边界框上和右边缘的地理点也可能包含在结果中。纬度的舍入误差小于 4.20 × 10^−8 度，经度的舍入误差小于 8.39 × 10^−8 度（大约 1 厘米）。

## 指定边界框

您可以通过提供以下任何一种顶点坐标组合来指定边界框：

- `top_right` 和 `bottom_left`
- `top_right` 和 `bottom_left`
- `top` , `left` , `bottom` , 和 `right`

以下示例展示了如何使用 `top` 、 `left` 、 `bottom` 和 `right` 坐标指定边界框：

```
GET testindex1/_search
{
  "query": {
    "bool": {
      "must": {
        "match_all": {}
      },
      "filter": {
        "geo_bounding_box": {
          "point": {
            "top": 75,
            "left": 28,
            "bottom": 73,
            "right": 41
          }
        }
      }
    }
  }
}
```

## 参数说明

地理边界框查询接受以下参数。

| 参数                | 数据类型 | 描述                                                                                                                                                             |
| ------------------- | -------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `_name`             | String   | 过滤器名称。可选。                                                                                                                                               |
| `validation_method` | String   | 验证方法。有效值有 `IGNORE_MALFORMED` （接受无效坐标的地理点）、 `COERCE` （尝试将坐标强制转换为有效值）和 `STRICT` （当坐标无效时返回错误）。默认为 `STRICT` 。 |
| `type`              | String   | 指定如何执行过滤器。有效值有 `indexed` （索引过滤器）和 `memory` （在内存中执行过滤器）。默认为 `memory` 。                                                      |
| `ignore_unmapped`   | Boolean  | 指定是否忽略未映射的字段。如果设置为 `true` ，则查询不会返回任何包含未映射字段的文档。如果设置为 `false` ，则在字段未映射时抛出异常。默认为 `false` 。           |
| `boost`             | Float    | 查询的相关性得分权重。默认为 `1.0`。                                                                                                                                          |

## 接受格式

您可以使用地理点字段类型接受的任何格式指定边界框顶点的坐标。

### 使用地理哈希指定边界框

如果您使用地理哈希来指定边界框，地理哈希被视为一个矩形。边界框的左上顶点对应于 `top_left` 地理哈希的左上顶点，边界框的右下顶点对应于 `bottom_right` 地理哈希的右下顶点。

以下示例展示了如何使用地理哈希来指定与前面示例相同的边界框：

```
GET testindex1/_search
{
  "query": {
    "bool": {
      "must": {
        "match_all": {}
      },
      "filter": {
        "geo_bounding_box": {
          "point": {
            "top_left": "ut7ftjkfxm34",
            "bottom_right": "uuvpkcprc4rc"
          }
        }
      }
    }
  }
}
```

要指定一个覆盖整个地理哈希区域的边界框，请将该地理哈希作为边界框的 `top_left` 和 `bottom_right` 参数提供：

```
GET testindex1/_search
{
  "query": {
    "bool": {
      "must": {
        "match_all": {}
      },
      "filter": {
        "geo_bounding_box": {
          "point": {
            "top_left": "ut",
            "bottom_right": "ut"
          }
        }
      }
    }
  }
}
```

