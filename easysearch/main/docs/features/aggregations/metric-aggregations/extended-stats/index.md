---
title: "扩展统计聚合（Extended Stats）"
date: 0001-01-01
summary: "扩展统计聚合 #  extended_stats 聚合是 stats 聚合的更全面版本。除了 stats 提供的基本统计指标外，extended_stats 还计算以下内容：
相关指南（先读这些） #     聚合基础
   聚合场景实践
  平方和
  方差
  总体方差
  抽样方差
  标准差
  总体标准差
  抽样标准差
  标准差界限： ** 上限 ** 下限 ** 总体上限 ** 总体下限 ** 抽样上限 ** 抽样下限
  其中标准差和方差是总体统计量；它们分别始终等于总体标准差和方差。
std_deviation_bounds 对象定义了一个范围，该范围在均值（默认为两个标准差）的上方和下方跨越指定的标准差数量。此对象始终包含在输出中，但仅对正态分布的数据才有意义。在解释这些值之前，请验证您的数据集是否遵循正态分布。
参数说明 #  extended_stats 聚合采用以下参数。
   参数 必需/可选 数据类型 描述     field 必需 String 返回扩展统计信息的字段名称。   sigma 可选 Double（非负） 计算 std_deviation_bounds 区间所使用的均值上下标准差的数量。默认值为 2 。   missing 可选 Numeric 分配给字段缺失实例的值。如果未提供，包含缺失值的文档将不会出现在扩展统计中。    参考样例 #  以下示例请求数据中 taxful_total_price 的扩展统计信息："
---


# 扩展统计聚合

`extended_stats` 聚合是 `stats` 聚合的更全面版本。除了 `stats` 提供的基本统计指标外，`extended_stats` 还计算以下内容：

## 相关指南（先读这些）

- [聚合基础]({{< relref "/docs/fundamentals/aggregations-data-analysis.md" >}})
- [聚合场景实践]({{< relref "/docs/features/aggregations/aggs-recipes.md" >}})

- 平方和
- 方差
- 总体方差
- 抽样方差
- 标准差
- 总体标准差
- 抽样标准差
- 标准差界限：
  \*\* 上限
  \*\* 下限
  \*\* 总体上限
  \*\* 总体下限
  \*\* 抽样上限
  \*\* 抽样下限

其中标准差和方差是总体统计量；它们分别始终等于总体标准差和方差。

`std_deviation_bounds` 对象定义了一个范围，该范围在均值（默认为两个标准差）的上方和下方跨越指定的标准差数量。此对象始终包含在输出中，但仅对正态分布的数据才有意义。在解释这些值之前，请验证您的数据集是否遵循正态分布。

## 参数说明

`extended_stats` 聚合采用以下参数。
| 参数 | 必需/可选 | 数据类型 | 描述 |
| --------- | --------- | -------- | ------------------------------------------------------------------- |
| `field` | 必需 | String | 返回扩展统计信息的字段名称。 |
| `sigma` | 可选 | Double（非负） | 计算 `std_deviation_bounds` 区间所使用的均值上下标准差的数量。默认值为 2 。 |
| `missing` | 可选 | Numeric | 分配给字段缺失实例的值。如果未提供，包含缺失值的文档将不会出现在扩展统计中。 |

## 参考样例

以下示例请求数据中 taxful_total_price 的扩展统计信息：

```
GET sample_data_ecommerce/_search
{
  "size": 0,
  "aggs": {
    "extended_stats_taxful_total_price": {
      "extended_stats": {
        "field": "taxful_total_price"
      }
    }
  }
}
```

## 返回内容

该内容包含 `taxful_total_price` 的扩展统计信息：

```
...
"aggregations" : {
  "extended_stats_taxful_total_price" : {
    "count" : 4675,
    "min" : 6.98828125,
    "max" : 2250.0,
    "avg" : 75.05542864304813,
    "sum" : 350884.12890625,
    "sum_of_squares" : 3.9367749294174194E7,
    "variance" : 2787.59157113862,
    "variance_population" : 2787.59157113862,
    "variance_sampling" : 2788.187974983536,
    "std_deviation" : 52.79764740155209,
    "std_deviation_population" : 52.79764740155209,
    "std_deviation_sampling" : 52.80329511482722,
    "std_deviation_bounds" : {
      "upper" : 180.6507234461523,
      "lower" : -30.53986616005605,
      "upper_population" : 180.6507234461523,
      "lower_population" : -30.53986616005605,
      "upper_sampling" : 180.66201887270256,
      "lower_sampling" : -30.551161586606312
    }
  }
 }
}
```

## 定义计算边界

可以通过将 `sigma` 参数设置为任何非负值来定义用于计算 `std_deviation_bounds` 区间的标准偏差数。

### 示例：定义计算边界

设置标准偏差的数量为 `std_deviation_bounds` 到 3 :

```
GET sample_data_ecommerce/_search
{
  "size": 0,
  "aggs": {
    "extended_stats_taxful_total_price": {
      "extended_stats": {
        "field": "taxful_total_price",
        "sigma": 3
      }
    }
  }
}
```

这会改变标准差界限：

```
{
...
  "aggregations": {
...
      "std_deviation_bounds": {
        "upper": 233.44837084770438,
        "lower": -83.33751356160813,
        "upper_population": 233.44837084770438,
        "lower_population": -83.33751356160813,
        "upper_sampling": 233.46531398752978,
        "lower_sampling": -83.35445670143353
      }
    }
  }
}
```

## 缺省值处理

可以为聚合字段的缺失实例分配一个值。有关更多信息，请参阅缺失聚合。
准备一个示例索引，通过导入以下文档：

```
POST _bulk
{ "create": { "_index": "students", "_id": "1" } }
{ "name": "John Doe", "gpa": 3.89, "grad_year": 2022}
{ "create": { "_index": "students", "_id": "2" } }
{ "name": "Jonathan Powers", "grad_year": 2025 }
{ "create": { "_index": "students", "_id": "3" } }
{ "name": "Jane Doe", "gpa": 3.52, "grad_year": 2024 }
```

### 示例：替换缺省值

计算 `extended_stats` ，将缺失的 GPA 字段替换为 0 ：

```
GET students/_search
{
  "size": 0,
  "aggs": {
    "extended_stats_gpa": {
      "extended_stats": {
        "field": "gpa",
        "missing": 0
      }
    }
  }
}

```

在返回内容中， gpa 的所有缺失值被替换为 0 ：

```
...
  "aggregations": {
    "extended_stats_gpa": {
      "count": 3,
      "min": 0,
      "max": 3.890000104904175,
      "avg": 2.4700000286102295,
      "sum": 7.4100000858306885,
      "sum_of_squares": 27.522500681877148,
      "variance": 3.0732667526245145,
      "variance_population": 3.0732667526245145,
      "variance_sampling": 4.609900128936772,
      "std_deviation": 1.7530735160353415,
      "std_deviation_population": 1.7530735160353415,
      "std_deviation_sampling": 2.147067797936705,
      "std_deviation_bounds": {
        "upper": 5.976147060680912,
        "lower": -1.0361470034604534,
        "upper_population": 5.976147060680912,
        "lower_population": -1.0361470034604534,
        "upper_sampling": 6.7641356244836395,
        "lower_sampling": -1.8241355672631805
      }
    }
  }
}
```

### 示例：忽略缺省值

计算 `extended_stats` 但不分配 `missing` 参数：

```
GET students/_search
{
  "size": 0,
  "aggs": {
    "extended_stats_gpa": {
      "extended_stats": {
        "field": "gpa"
      }
    }
  }
}

```

计算扩展统计信息，忽略包含缺失字段值的文档（默认行为）：

```
...
  "aggregations": {
    "extended_stats_gpa": {
      "count": 2,
      "min": 3.5199999809265137,
      "max": 3.890000104904175,
      "avg": 3.7050000429153442,
      "sum": 7.4100000858306885,
      "sum_of_squares": 27.522500681877148,
      "variance": 0.03422502293587115,
      "variance_population": 0.03422502293587115,
      "variance_sampling": 0.0684500458717423,
      "std_deviation": 0.18500006198883057,
      "std_deviation_population": 0.18500006198883057,
      "std_deviation_sampling": 0.2616295967044675,
      "std_deviation_bounds": {
        "upper": 4.075000166893005,
        "lower": 3.334999918937683,
        "upper_population": 4.075000166893005,
        "lower_population": 3.334999918937683,
        "upper_sampling": 4.228259236324279,
        "lower_sampling": 3.1817408495064092
      }
    }
  }
}
```

包含缺失 GPA 值的文档在此计算中被省略。注意 `count` 中的差异。

