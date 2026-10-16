---
title: "Geo Shape 查询"
date: 0001-01-01
description: "形状空间关系查询：INTERSECTS、DISJOINT、WITHIN、CONTAINS。"
summary: "Geo Shape 查询 #  地理形状查询用于搜索 geo_point 或 geo_shape 字段的文档。可以使用查询中内联定义的形状，或引用预索引的形状来过滤文档。
空间关系 #  当您向地理形状查询提供地理形状时，文档中的地理点和地理形状字段将使用以下空间关系与提供的形状进行匹配。
   关系 描述 支持地理字段类型     INTERSECTS （默认）匹配与查询中提供的形状相交的 geopoint 或 geoshape 的文档。 geo_point, geo_shape   DISJOINT 匹配与查询中提供的形状不相交的 geoshape 的文档。 geo_shape   WITHIN 匹配完全位于查询中提供的形状内的 geoshape 的文档。 geo_shape   CONTAINS 匹配完全包含查询中提供的形状的文档。 geo_shape    在 geoshape 查询中定义形状 #  您可以在 geoshape 查询中通过在查询时提供新的形状定义或引用另一个索引中预先索引的形状名称来定义形状以过滤文档。
使用新的形状定义 #  为了向地理形状查询提供新的形状，请在 geo_shape 字段中定义它。您必须以 GeoJSON 格式定义地理形状。"
---


# Geo Shape 查询

地理形状查询用于搜索 `geo_point` 或 `geo_shape` 字段的文档。可以使用查询中内联定义的形状，或引用预索引的形状来过滤文档。

## 空间关系

当您向地理形状查询提供地理形状时，文档中的地理点和地理形状字段将使用以下空间关系与提供的形状进行匹配。

| 关系         | 描述                                                               | 支持地理字段类型         |
| ------------ | ------------------------------------------------------------------ | ------------------------ |
| `INTERSECTS` | （默认）匹配与查询中提供的形状相交的 geopoint 或 geoshape 的文档。 | `geo_point`, `geo_shape` |
| `DISJOINT`   | 匹配与查询中提供的形状不相交的 geoshape 的文档。                   | `geo_shape`              |
| `WITHIN`     | 匹配完全位于查询中提供的形状内的 geoshape 的文档。                 | `geo_shape`              |
| `CONTAINS`   | 匹配完全包含查询中提供的形状的文档。                               | `geo_shape`              |

## 在 geoshape 查询中定义形状

您可以在 geoshape 查询中通过在查询时提供新的形状定义或引用另一个索引中预先索引的形状名称来定义形状以过滤文档。

## 使用新的形状定义

为了向地理形状查询提供新的形状，请在 `geo_shape` 字段中定义它。您必须以 GeoJSON 格式定义地理形状。

以下示例说明了搜索包含在查询时定义的地理形状的文档。

首先，创建一个索引并将 `location` 字段映射为 `geo_shape` ：

```
PUT /testindex
{
  "mappings": {
    "properties": {
      "location": {
        "type": "geo_shape"
      }
    }
  }
}
```

索引包含一个点和另一个包含多边形的文档：

```
PUT testindex/_doc/1
{
  "location": {
    "type": "point",
    "coordinates": [ 73.0515, 41.5582 ]
  }
}

PUT testindex/_doc/2
{
  "location": {
    "type": "polygon",
    "coordinates": [
      [
        [
          73.0515,
          41.5582
        ],
        [
          72.6506,
          41.5623
        ],
        [
          72.6734,
          41.7658
        ],
        [
          73.0515,
          41.5582
        ]
      ]
    ]
  }
}
```

最后，定义一个 geoshape 以过滤文档。以下各节展示了在查询中提供各种 geoshape 的方法。有关各种 geoshape 格式的更多信息，请参阅 Geoshape 字段类型。

#### Envelope 边界矩形

一个 `envelope` 是 `[[minLon, maxLat], [maxLon, minLat]]` 格式中的一个边界矩形。搜索包含与提供的边界矩形相交的 geoshape 字段文档：

```
GET /testindex/_search
{
  "query": {
    "geo_shape": {
      "location": {
        "shape": {
          "type": "envelope",
          "coordinates": [
            [
              71.0589,
              42.3601
            ],
            [
              74.006,
              40.7128
            ]
          ]
        },
        "relation": "WITHIN"
      }
    }
  }
}
```

返回内容包含两个文档：

```
{
  "took": 5,
  "timed_out": false,
  "_shards": {
    "total": 1,
    "successful": 1,
    "skipped": 0,
    "failed": 0
  },
  "hits": {
    "total": {
      "value": 2,
      "relation": "eq"
    },
    "max_score": 0,
    "hits": [
      {
        "_index": "testindex",
        "_id": "1",
        "_score": 0,
        "_source": {
          "location": {
            "type": "point",
            "coordinates": [
              73.0515,
              41.5582
            ]
          }
        }
      },
      {
        "_index": "testindex",
        "_id": "2",
        "_score": 0,
        "_source": {
          "location": {
            "type": "polygon",
            "coordinates": [
              [
                [
                  73.0515,
                  41.5582
                ],
                [
                  72.6506,
                  41.5623
                ],
                [
                  72.6734,
                  41.7658
                ],
                [
                  73.0515,
                  41.5582
                ]
              ]
            ]
          }
        }
      }
    ]
  }
}

```

#### Point 点位

搜索包含提供的点的地理形状字段的文档：

```
GET /testindex/_search
{
  "query": {
    "geo_shape": {
      "location": {
        "shape": {
          "type": "point",
          "coordinates": [
            72.8000,
            41.6300
          ]
        },
        "relation": "CONTAINS"
      }
    }
  }
}
```

#### Linestring 线字符串

搜索与提供的线字符串不交叉的地理形状字段文档：

```
GET /testindex/_search
{
  "query": {
    "geo_shape": {
      "location": {
        "shape": {
          "type": "linestring",
          "coordinates": [[74.0060, 40.7128], [71.0589, 42.3601]]
        },
        "relation": "DISJOINT"
      }
    }
  }
}
```

> 线字符串地理形状查询不支持 WITHIN 关系。

#### Multipolygon 多边形

搜索地理形状字段在提供的多边形内的文档：

```
GET /testindex/_search
{
  "query": {
    "geo_shape": {
      "location": {
        "shape": {
          "type" : "multipolygon",
          "coordinates" : [
          [
            [
              [74.0060, 40.7128],
              [73.7562, 42.6526],
              [71.0589, 42.3601],
              [74.0060, 40.7128]
            ],
            [
              [73.0515, 41.5582],
              [72.6506, 41.5623],
              [72.6734, 41.7658],
              [73.0515, 41.5582]
            ]
          ],
          [
            [
              [73.9146, 40.8252],
              [73.8871, 41.0389],
              [73.6853, 40.9747],
              [73.9146, 40.8252]
            ]
          ]
        ]
        },
        "relation": "WITHIN"
      }
    }
  }
}
```

#### Geometry collection 几何集合

搜索地理形状字段在提供的多边形内的文档：

```
GET /testindex/_search
{
  "query": {
    "geo_shape": {
      "location": {
        "shape": {
          "type": "geometrycollection",
          "geometries": [
            {
              "type": "polygon",
              "coordinates": [[
                [74.0060, 40.7128],
                [73.7562, 42.6526],
                [71.0589, 42.3601],
                [74.0060, 40.7128]
              ]]
            },
            {
              "type": "polygon",
              "coordinates": [[
                [73.0515, 41.5582],
                [72.6506, 41.5623],
                [72.6734, 41.7658],
                [73.0515, 41.5582]
              ]]
            }
          ]
        },
        "relation": "WITHIN"
      }
    }
  }
}
```

> Geoshape 查询中，如果几何集合包含线字符串或多线字符串，则不支持 WITHIN 关系。

## 使用预索引的形状定义

在构建 geoshape 查询时，您还可以引用另一个索引中预先索引的形状的名称。使用此方法，您可以在索引时定义 geoshape，并在搜索时通过名称引用它。

您可以使用 GeoJSON 或 WKT 格式定义预索引的 geoshape。有关各种 geoshape 格式的更多信息，请参阅 Geoshape 字段类型。

`indexed_shape` 对象支持以下参数。

| 参数      | 必填/可选 | 描述                                            |
| --------- | --------- | ----------------------------------------------- |
| `id`      | 必填      | 包含预索引形状的文档的文档 ID。                 |
| `index`   | 可选      | 包含预索引形状的索引名称。默认为 `shapes` 。    |
| `path`    | 可选      | 包含预索引形状路径的字段名称。默认为 `shape` 。 |
| `routing` | 可选      | 包含预索引形状的文档的路由。                    |

以下示例说明了如何引用另一个索引中预索引的形状的名称。在此示例中，索引 `pre-indexed-shapes` 包含定义边界的形状，索引 `testindex` 包含与这些边界进行校验的形状。

首先，创建 `pre-indexed-shapes` 索引并将此索引的 `boundaries` 字段映射为 `geo_shape` :

```
PUT /pre-indexed-shapes
{
  "mappings": {
    "properties": {
      "boundaries": {
        "type": "geo_shape",
        "orientation" : "left"
      }
    }
  }
}
```

关于为多边形指定不同的顶点方向，请参阅《多边形》。

将指定搜索边界的多边形索引到 `pre-indexed-shapes` 索引中。多边形的 ID 是 `search_triangle` 。在此示例中，您将以 `WKT` 格式索引多边形：

```
PUT /pre-indexed-shapes/_doc/search_triangle
{
  "boundaries":
    "POLYGON ((74.0060 40.7128, 71.0589 42.3601, 73.7562 42.6526, 74.0060 40.7128))"
}
```

如果您尚未这样做，请将包含一个点的一个文档和包含一个多边形的一个文档索引到 `testindex` 索引中：

```
PUT /testindex/_doc/1
{
  "location": {
    "type": "point",
    "coordinates": [ 73.0515, 41.5582 ]
  }
}

PUT /testindex/_doc/2
{
  "location": {
    "type": "polygon",
    "coordinates": [
      [
        [
          73.0515,
          41.5582
        ],
        [
          72.6506,
          41.5623
        ],
        [
          72.6734,
          41.7658
        ],
        [
          73.0515,
          41.5582
        ]
      ]
    ]
  }
}
```

搜索地理形状在 `search_triangle` 内的文档：

```
GET /testindex/_search
{
  "query": {
    "bool": {
      "must": {
        "match_all": {}
      },
      "filter": {
        "geo_shape": {
          "location": {
            "indexed_shape": {
              "index": "pre-indexed-shapes",
              "id": "search_triangle",
              "path": "boundaries"
            },
            "relation": "WITHIN"
          }
        }
      }
    }
  }
}
```

返回内容包含两个文档：

```
{
  "took": 11,
  "timed_out": false,
  "_shards": {
    "total": 1,
    "successful": 1,
    "skipped": 0,
    "failed": 0
  },
  "hits": {
    "total": {
      "value": 2,
      "relation": "eq"
    },
    "max_score": 1,
    "hits": [
      {
        "_index": "testindex",
        "_id": "1",
        "_score": 1,
        "_source": {
          "location": {
            "type": "point",
            "coordinates": [
              73.0515,
              41.5582
            ]
          }
        }
      },
      {
        "_index": "testindex",
        "_id": "2",
        "_score": 1,
        "_source": {
          "location": {
            "type": "polygon",
            "coordinates": [
              [
                [
                  73.0515,
                  41.5582
                ],
                [
                  72.6506,
                  41.5623
                ],
                [
                  72.6734,
                  41.7658
                ],
                [
                  73.0515,
                  41.5582
                ]
              ]
            ]
          }
        }
      }
    ]
  }
}

```

## 查询地理点

您还可以使用地理形状查询来搜索包含地理点的文档。

> 地理点字段的地理形状查询仅支持默认的 INTERSECTS 空间关系，因此您不需要提供 relation 参数。

> 地理点字段的地理形状查询不支持以下地理形状：
>
> - Points 点
> - Linestrings 线字符串
> - Multipoints 多点
> - Multilinestrings 多线字符串
> - Geometry collections containing one of the preceding geoshape types
> - 包含前述地理形状类型之一的几何集合

创建一个映射，其中 `location` 是 `geo_point` ：

```
PUT /testindex1
{
  "mappings": {
    "properties": {
      "location": {
        "type": "geo_point"
      }
    }
  }
}
```

将两个点索引到索引中。在这个例子中，您将提供地理坐标点作为字符串：

```
PUT /testindex1/_doc/1
{
  "location": "41.5623, 72.6506"
}

PUT /testindex1/_doc/2
{
  "location": "76.0254, 39.2467"
}
```

搜索与提供的多边形相交的地理点：

```
GET /testindex1/_search
{
  "query": {
    "geo_shape": {
      "location": {
        "shape": {
          "type": "polygon",
          "coordinates": [
            [
              [74.0060, 40.7128],
              [73.7562, 42.6526],
              [71.0589, 42.3601],
              [74.0060, 40.7128]
            ]
          ]
        }
      }
    }
  }
}
```

返回文档 1：

```
{
  "took": 21,
  "timed_out": false,
  "_shards": {
    "total": 1,
    "successful": 1,
    "skipped": 0,
    "failed": 0
  },
  "hits": {
    "total": {
      "value": 1,
      "relation": "eq"
    },
    "max_score": 0,
    "hits": [
      {
        "_index": "testindex1",
        "_id": "1",
        "_score": 0,
        "_source": {
          "location": "41.5623, 72.6506"
        }
      }
    ]
  }
}

```

请注意，当您索引地理点时，您指定了它们的坐标为 `"latitude, longitude"` 格式。当您搜索匹配的文档时，坐标数组为 `[longitude, latitude]` 格式。因此，文档 1 在结果中返回，但文档 2 则没有。

## 参数说明

Geoshape 查询接受以下参数。

| 参数              | 数据类型 | 描述                                                                                                                                                 |
| ----------------- | -------- | ---------------------------------------------------------------------------------------------------------------------------------------------------- |
| `relation`        | String   | 空间关系。有效值为 `INTERSECTS`（默认）、`DISJOINT`、`WITHIN`、`CONTAINS`。                                                                          |
| `ignore_unmapped` | Boolean  | 指定是否忽略未映射的字段。如果设置为 `true`，则查询不返回包含未映射字段的任何文档。如果设置为 `false`，则在字段未映射时抛出异常。可选。默认为 `false`。 |
| `boost`           | Float    | 查询的相关性得分权重。默认为 `1.0`。                                                                                                                  |

