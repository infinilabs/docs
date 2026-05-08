---
title: "热门匹配聚合（Top Hits）"
date: 0001-01-01
summary: "热门匹配聚合 #  top_hits 聚合是一种多值指标聚合，它根据聚合字段的相关性评分对匹配文档进行排名。
相关指南（先读这些） #    聚合基础  聚合场景实践  您可以指定以下选项：
 from : 命中的起始位置。 size : 返回命中的最大数量。默认值为 3。 sort : 匹配命中的排序方式。默认情况下，命中的排序依据聚合查询的相关性得分。  以下示例返回数据中的前 5 个产品：
GET sample_data_ecommerce/_search { &#34;size&#34;: 0, &#34;aggs&#34;: { &#34;top_hits_products&#34;: { &#34;top_hits&#34;: { &#34;size&#34;: 5 } } } } 返回内容
... &#34;aggregations&#34; : { &#34;top_hits_products&#34; : { &#34;hits&#34; : { &#34;total&#34; : { &#34;value&#34; : 4675, &#34;relation&#34; : &#34;eq&#34; }, &#34;max_score&#34; : 1."
---


# 热门匹配聚合

`top_hits` 聚合是一种多值指标聚合，它根据聚合字段的相关性评分对匹配文档进行排名。

## 相关指南（先读这些）

- [聚合基础]({{< relref "/docs/fundamentals/aggregations-data-analysis.md" >}})
- [聚合场景实践]({{< relref "/docs/features/aggregations/aggs-recipes.md" >}})

您可以指定以下选项：

- from : 命中的起始位置。
- size : 返回命中的最大数量。默认值为 3。
- sort : 匹配命中的排序方式。默认情况下，命中的排序依据聚合查询的相关性得分。

以下示例返回数据中的前 5 个产品：

```
GET sample_data_ecommerce/_search
{
  "size": 0,
  "aggs": {
    "top_hits_products": {
      "top_hits": {
        "size": 5
      }
    }
  }
}
```

返回内容

```
...
"aggregations" : {
  "top_hits_products" : {
    "hits" : {
      "total" : {
        "value" : 4675,
        "relation" : "eq"
      },
      "max_score" : 1.0,
      "hits" : [
        {
          "_index" : "sample_data_ecommerce",
          "_type" : "_doc",
          "_id" : "glMlwXcBQVLeQPrkHPtI",
          "_score" : 1.0,
          "_source" : {
            "category" : [
              "Women's Accessories",
              "Women's Clothing"
            ],
            "currency" : "EUR",
            "customer_first_name" : "rania",
            "customer_full_name" : "rania Evans",
            "customer_gender" : "FEMALE",
            "customer_id" : 24,
            "customer_last_name" : "Evans",
            "customer_phone" : "",
            "day_of_week" : "Sunday",
            "day_of_week_i" : 6,
            "email" : "rania@evans-family.zzz",
            "manufacturer" : [
              "Tigress Enterprises"
            ],
            "order_date" : "2021-02-28T14:16:48+00:00",
            "order_id" : 583581,
            "products" : [
              {
                "base_price" : 10.99,
                "discount_percentage" : 0,
                "quantity" : 1,
                "manufacturer" : "Tigress Enterprises",
                "tax_amount" : 0,
                "product_id" : 19024,
                "category" : "Women's Accessories",
                "sku" : "ZO0082400824",
                "taxless_price" : 10.99,
                "unit_discount_amount" : 0,
                "min_price" : 5.17,
                "_id" : "sold_product_583581_19024",
                "discount_amount" : 0,
                "created_on" : "2016-12-25T14:16:48+00:00",
                "product_name" : "Snood - white/grey/peach",
                "price" : 10.99,
                "taxful_price" : 10.99,
                "base_unit_price" : 10.99
              },
              {
                "base_price" : 32.99,
                "discount_percentage" : 0,
                "quantity" : 1,
                "manufacturer" : "Tigress Enterprises",
                "tax_amount" : 0,
                "product_id" : 19260,
                "category" : "Women's Clothing",
                "sku" : "ZO0071900719",
                "taxless_price" : 32.99,
                "unit_discount_amount" : 0,
                "min_price" : 17.15,
                "_id" : "sold_product_583581_19260",
                "discount_amount" : 0,
                "created_on" : "2016-12-25T14:16:48+00:00",
                "product_name" : "Cardigan - grey",
                "price" : 32.99,
                "taxful_price" : 32.99,
                "base_unit_price" : 32.99
              }
            ],
            "sku" : [
              "ZO0082400824",
              "ZO0071900719"
            ],
            "taxful_total_price" : 43.98,
            "taxless_total_price" : 43.98,
            "total_quantity" : 2,
            "total_unique_products" : 2,
            "type" : "order",
            "user" : "rani",
            "geoip" : {
              "country_iso_code" : "EG",
              "location" : {
                "lon" : 31.3,
                "lat" : 30.1
              },
              "region_name" : "Cairo Governorate",
              "continent_name" : "Africa",
              "city_name" : "Cairo"
            },
            "event" : {
              "dataset" : "sample_ecommerce"
            }
          }
          ...
        }
      ]
    }
  }
 }
}
```

