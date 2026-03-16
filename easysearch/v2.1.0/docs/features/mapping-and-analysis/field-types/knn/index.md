---
title: "向量字段类型（K-NN）"
date: 0001-01-01
summary: "K-NN 向量字段类型 #  相关指南（先读这些） #    向量搜索  向量字段建模  映射基础  关于向量 #  在索引文档和运行查询时都需要指定向量类型。在这两种情况下，您都使用相同的 JSON 结构来定义向量类型。每个向量类型还有一个简写形式，这在使用不支持嵌套文档的工具时会很方便。以下示例展示了如何在索引向量时指定它们。
knn_dense_float_vector 密集向量类型 #  假设您已经定义了一个映射，其中 my_vec 的类型为 knn_dense_float_vector。
POST /my-index/_doc { &#34;my_vec&#34;: { &#34;values&#34;: [0.1, 0.2, 0.3, ...] # 1 } } POST /my-index/_doc { &#34;my_vec&#34;: [0.1, 0.2, 0.3, ...] # 2 } 说明 #  1	向量中所有浮点值的 JSON 列表。长度应与映射中的dims匹配。 2	#1 的简写形式。
knn_sparse_bool_vector 稀疏向量类型 #  假设您已经定义了一个映射，其中 my_vec 的类型为 knn_sparse_bool_vector。"
---


# K-NN 向量字段类型

## 相关指南（先读这些）

- [向量搜索]({{< relref "/docs/features/vector-search/_index.md" >}})
- [向量字段建模]({{< relref "/docs/best-practices/data-modeling/vector-fields.md" >}})
- [映射基础]({{< relref "/docs/features/mapping-and-analysis/mapping-basics.md" >}})

## 关于向量

在索引文档和运行查询时都需要指定向量类型。在这两种情况下，您都使用相同的 JSON 结构来定义向量类型。每个向量类型还有一个简写形式，这在使用不支持嵌套文档的工具时会很方便。以下示例展示了如何在索引向量时指定它们。

### knn_dense_float_vector 密集向量类型

假设您已经定义了一个映射，其中 my_vec 的类型为 knn_dense_float_vector。

```json
POST /my-index/_doc
{
    "my_vec": {
        "values": [0.1, 0.2, 0.3, ...]    # 1
    }
}
POST /my-index/_doc
{
    "my_vec": [0.1, 0.2, 0.3, ...]        # 2
}
```

#### 说明
1	向量中所有浮点值的 JSON 列表。长度应与映射中的`dims`匹配。
2	#1 的简写形式。

### knn_sparse_bool_vector 稀疏向量类型

假设您已经定义了一个映射，其中 my_vec 的类型为 knn_sparse_bool_vector。

```json
POST /my-index/_doc
{
    "my_vec": {
       "true_indices": [1, 3, 5, ...],   # 1
       "total_indices": 100,             # 2
    }
}
POST /my-index/_doc
{
    "my_vec": [[1, 3, 5, ...], 100]      # 3
}
```

#### 说明
1	向量中为 `true` 的索引的 JSON 列表。
2	索引的向量总数。这应该与映射中的`dims`匹配。
3	#1 和 #2 的简写形式。一个包含两个项目的列表，第一个项目是 `true_indices`，第二个是 `total_indices`。

## 映射

在索引向量数据之前，您需要先定义一个映射，指定向量数据类型、索引模型和模型参数。这决定了索引向量支持哪些查询。

###  基本结构

基本映射结构如下：

```json
PUT /my-index/_mapping
{
  "properties": {                               # 1
    "my_vec": {                                 # 2 
      "type": "knn_sparse_bool_vector",         # 3
      "knn": {                            # 4
        "dims": 100,                            # 5
        "model": "exact",                       # 6
        ...                                     # 7
      }
    }
  }
}
```

#### 说明
1	文档字段的字典。与 PUT Mapping API 相同。
2	向量的字段名称。
3	要存储的向量类型。
4	knn 的设置。
5	向量的维度。存储在此字段 `my_vec` 中的所有向量必须具有相同的维度。
6	模型类型。这和模型参数将决定您可以运行什么样的搜索。请参见下面的模型部分。
7	额外的模型参数。请参见下面的模型部分。

### knn_sparse_bool_vector 数据类型

这种类型针对每个索引为 `true` 或 `false` 的向量进行了优化,大部分的情况为 `false`。例如，您可以表示文档的词袋编码，其中每个索引对应词汇表中的一个词，而任何单个文档只包含所有词的很小一部分。在内部，Knn 通过只存储 true 的数据来节省空间。

```json
PUT /my-index/_mapping
{
    "properties": {
        "my_vec": {
            "type": "knn_sparse_bool_vector",        # 1
            "knn": {
                "dims": 25000,                       # 2
                ...                                  # 3
            }
        }
    }
}
```

####	说明
1	类型名称。
2	向量的维度。
3	额外的模型参数。请参见下面的模型部分。

### knn_dense_float_vector 数据类型

这种类型针对每个向量都是浮点数、所有索引都有值且维度通常不超过约 1000 的向量进行了优化。例如，您可以存储词嵌入或图像向量。在内部，Knn 使用 Java Float 类型来存储值。

```json
PUT /my-index/_mapping
{
    "properties": {
        "my_vec": {
            "type": "knn_dense_float_vector",        # 1
            "knn": {
                "dims": 100,                         # 2
                ...                                  # 3
            }
        }
    }
}
```

#### 说明
1	类型名称。
2	向量的维度。不应超过几千。如果超过，请考虑进行某种维度降低。
3	额外的模型参数。请参见下面的模型部分。

### 特定模型映射

特定模型映射允许您运行特定模型的搜索。这些搜索不利用任何索引结构，运行时间复杂度为 `O(n^2)`，其中 `n` 是文档总数。

使用此配置时，您不需要提供任何 `"model": "..."` 值或任何模型参数。

```json
PUT /my-index/_mapping
{
    "properties": {
        "my_vec": {
            "type": "knn_(dense_float | sparse_bool)_vector",        # 1
            "knn": {
                "dims": 100,                                         # 2
            }
        }
    }
}
```

#### 说明
1	向量数据类型。支持 dense float 和 sparse bool 两种类型
2	向量维度。

### Jaccard LSH 映射

使用 [Minhash 算法](https://en.wikipedia.org/wiki/MinHash)对稀疏布尔向量进行哈希和存储，以支持近似 Jaccard 相似度查询。

该实现受到 [《Mining Massive Datasets》](http://www.mmds.org/)第 3 章、Spark MinHash 实现、[tdebatty/java-LSH](https://github.com/tdebatty/java-LSH) Github 项目和 [Minhash for Dummies](http://matthewcasperson.blogspot.com/2013/11/minhash-for-dummies.html) 博客文章的影响。

```json
PUT /my-index/_mapping
{
    "properties": {
        "my_vec": {
            "type": "knn_sparse_bool_vector",       # 1
            "knn": {
                "dims": 25000,                      # 2
                "model": "lsh",                     # 3
                "similarity": "jaccard",            # 4
                "L": 99,                            # 5
                "k": 1                              # 6
            }
        }
    }
}
```

#### 说明
1	向量数据类型。必须是稀疏布尔向量。
2	向量维度。
3	模型类型。
4	相似度度量。
5	哈希表的数量。通常增加这个值会提高召回率。
6	组合形成单个哈希值的哈希函数数量。通常，增加这个值会提高精确度。

### 汉明 LSH 映射

使用[位采样算法](http://mlwiki.org/index.php/Bit_Sampling_LSH)对稀疏布尔向量进行哈希和存储，以支持近似汉明相似度查询。

与标准位采样方法的唯一区别是它采样并组合 `k` 个位来形成单个哈希值。例如，如果设置 `L = 100`，`k = 3`，它会从向量中采样 `100 * 3 = 300` 个位，并将每 3 个位连接起来形成每个哈希值，总共生成 100 个哈希值。

```json
PUT /my-index/_mapping
{
    "properties": {
        "my_vec": {
            "type": "knn_sparse_bool_vector",       # 1
            "knn": {
                "dims": 25000,                      # 2
                "model": "lsh",                     # 3
                "similarity": "hamming",            # 4
                "L": 99,                            # 5
                "k": 2                              # 6
            }
        }
    }
}
```

#### 说明
1	向量数据类型。必须是稀疏布尔向量。
2	向量维度。
3	模型类型。
4	相似度度量。
5	哈希表的数量。通常增加这个值会提高召回率。
6	组合形成单个哈希值的哈希函数数量。通常增加这个值会提高精确度。

### 余弦 LSH 映射

使用[随机投影算法](https://en.wikipedia.org/wiki/Locality-sensitive_hashing#Random_projection)对密集浮点向量进行哈希和存储，以支持近似余弦相似度查询。

参考[ Mining Massive Datasets](https://en.wikipedia.org/wiki/Locality-sensitive_hashing#Random_projection)第三章

```json
PUT /my-index/_mapping
{
    "properties": {
        "my_vec": {
            "type": "knn_dense_float_vector",       # 1
            "knn": {
                "dims": 100,                        # 2
                "model": "lsh",                     # 3
                "similarity": "cosine",             # 4
                "L": 99,                            # 5
                "k": 1                              # 6
            }
        }
    }
}
```

#### 说明
1	向量数据类型。必须是密集浮点向量。
2	向量维度。
3	模型类型。
4	相似度度量。
5	哈希表的数量。通常增加这个值会提高召回率。
6	组合形成单个哈希值的哈希函数数量。通常，增加这个值会提高精确度。

### L2 LSH 映射

使用[稳定分布方法](https://en.wikipedia.org/wiki/Locality-sensitive_hashing#Stable_distributions)对密集浮点向量进行哈希和存储，以支持近似 L2（欧几里得）相似度查询。

参考[ Mining Massive Datasets](https://en.wikipedia.org/wiki/Locality-sensitive_hashing#Random_projection)第三章
```json
PUT /my-index/_mapping
{
    "properties": {
        "my_vec": {
            "type": "knn_dense_float_vector",       # 1
            "knn": {
                "dims": 100,                        # 2
                "model": "lsh",                     # 3
                "similarity": "l2",                 # 4
                "L": 99,                            # 5
                "k": 1,                             # 6
                "w": 3                              # 7
            }
        }
    }
}
```

#### 说明
1	向量数据类型。必须是密集浮点向量。
2	向量维度。
3	模型类型。
4	相似度度量。
5	哈希表的数量。通常增加这个值会提高召回率。
6	组合形成单个哈希值的哈希函数数量。通常增加这个值会提高精确度。
7	整数桶宽度。这决定了两个向量在投影到第三个公共向量上时必须多么接近，才能共享一个哈希值。典型值是低位整数。

### 排列 LSH 映射

使用 Amato 等人在[《Large-Scale Image Retrieval with Elasticsearch》](https://dl.acm.org/doi/10.1145/3209978.3210089)中描述的模型。

该模型通过向量中绝对值最大的 `k` 个索引（向量中的位置）来描述向量。直觉是，每个索引对应于某个潜在概念，绝对值高的索引比绝对值低的索引携带更多关于其各自概念的信息。该方法的研究主要集中在余弦相似度上，尽管实现也支持 L1 和 L2。

#### 例子

向量 `[10, -2, 0, 99, 0.1, -8, 42, -13, 6, 0.1]` 中 `k = 4` 的索引为 `[4, 7, -8, 1]`。索引是 1 索引的，负值的索引被否定（因此 -8）。索引可以根据其排名可选地重复。在本例中，索引将被重复 `[4, 4, 4, 4, 7, 7, 7, -8, -8, 1]`。索引 4 具有最高的绝对值，因此它被重复 `k - 0 = 4` 次。索引 7 具有第二高的绝对值，因此它被重复 `k - 1 = 3` 次，依此类推。搜索算法将得分计算为存储向量表示和查询向量表示的交集的大小。因此，对于表示为 `[2, 2, 2, 2, 7, 7, 7, 4, 4, 5]` 的查询向量，交集为 `[7, 7, 7, 4, 4]`，得分为 5。在一些实验中，重复实际上降低了召回率，因此建议您尝试使用和不使用重复。

```json
PUT /my-index/_mapping
{
    "properties": {
        "my_vec": {
            "type": "knn_dense_float_vector",       # 1
            "knn": {
                "dims": 100,                        # 2
                "model": "permutation_lsh",         # 3
                "k": 10,                            # 4
                "repeating": true                   # 5
            }
        }
    }
}
```

#### 说明
1	向量数据类型。必须是密集浮点向量。
2	向量维度。
3	模型类型。
4	选择的顶部索引数量。
5	是否根据其排名重复索引。请参见上面的重复注释。
