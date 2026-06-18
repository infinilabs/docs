---
title: "ICU 脚本转换过滤器（ICU Transform）"
date: 0001-01-01
summary: "ICU 脚本转换过滤器 #  icu_transform 词元过滤器使用 ICU 的 Transliterator 引擎将文本从一种脚本转写为另一种脚本。
前提条件 #  需要安装 analysis-icu 插件：
bin/easysearch-plugin install analysis-icu 功能说明 #  此过滤器可以实现：
 脚本转写：中文→拉丁、西里尔→拉丁、阿拉伯→拉丁等 大小写转换：Upper、Lower、Title Unicode 归一化：NFC、NFD、NFKC、NFKD 自定义转换规则  使用示例 #  西里尔字母转拉丁字母 #  PUT my-transliterate-index { &#34;settings&#34;: { &#34;analysis&#34;: { &#34;filter&#34;: { &#34;cyrillic_to_latin&#34;: { &#34;type&#34;: &#34;icu_transform&#34;, &#34;id&#34;: &#34;Cyrillic-Latin&#34; } }, &#34;analyzer&#34;: { &#34;my_translit&#34;: { &#34;type&#34;: &#34;custom&#34;, &#34;tokenizer&#34;: &#34;standard&#34;, &#34;filter&#34;: [&#34;cyrillic_to_latin&#34;] } } } } } 中文转拼音 #  PUT my-pinyin-index { &#34;settings&#34;: { &#34;analysis&#34;: { &#34;filter&#34;: { &#34;han_to_latin&#34;: { &#34;type&#34;: &#34;icu_transform&#34;, &#34;id&#34;: &#34;Han-Latin&#34; } } } } } 链式转换 #  PUT my-chain-index { &#34;settings&#34;: { &#34;analysis&#34;: { &#34;filter&#34;: { &#34;to_ascii&#34;: { &#34;type&#34;: &#34;icu_transform&#34;, &#34;id&#34;: &#34;Cyrillic-Latin; NFD; [:Nonspacing Mark:] Remove; NFC&#34; } } } } } 测试效果 #  GET /_analyze { &#34;tokenizer&#34;: &#34;standard&#34;, &#34;filter&#34;: [{&#34;type&#34;: &#34;icu_transform&#34;, &#34;id&#34;: &#34;Cyrillic-Latin&#34;}], &#34;text&#34;: &#34;Москва&#34; } 响应：Moskva"
---


# ICU 脚本转换过滤器

`icu_transform` 词元过滤器使用 ICU 的 Transliterator 引擎将文本从一种脚本转写为另一种脚本。

## 前提条件

需要安装 analysis-icu 插件：

```bash
bin/easysearch-plugin install analysis-icu
```

## 功能说明

此过滤器可以实现：

- **脚本转写**：中文→拉丁、西里尔→拉丁、阿拉伯→拉丁等
- **大小写转换**：Upper、Lower、Title
- **Unicode 归一化**：NFC、NFD、NFKC、NFKD
- **自定义转换规则**

## 使用示例

### 西里尔字母转拉丁字母

```json
PUT my-transliterate-index
{
  "settings": {
    "analysis": {
      "filter": {
        "cyrillic_to_latin": {
          "type": "icu_transform",
          "id": "Cyrillic-Latin"
        }
      },
      "analyzer": {
        "my_translit": {
          "type": "custom",
          "tokenizer": "standard",
          "filter": ["cyrillic_to_latin"]
        }
      }
    }
  }
}
```

### 中文转拼音

```json
PUT my-pinyin-index
{
  "settings": {
    "analysis": {
      "filter": {
        "han_to_latin": {
          "type": "icu_transform",
          "id": "Han-Latin"
        }
      }
    }
  }
}
```

### 链式转换

```json
PUT my-chain-index
{
  "settings": {
    "analysis": {
      "filter": {
        "to_ascii": {
          "type": "icu_transform",
          "id": "Cyrillic-Latin; NFD; [:Nonspacing Mark:] Remove; NFC"
        }
      }
    }
  }
}
```

### 测试效果

```json
GET /_analyze
{
  "tokenizer": "standard",
  "filter": [{"type": "icu_transform", "id": "Cyrillic-Latin"}],
  "text": "Москва"
}
```

响应：`Moskva`

## 参数

| 参数 | 类型 | 必填 | 说明 |
|------|------|------|------|
| `id` | string | ✅ | ICU 转换 ID，如 `Cyrillic-Latin`、`Han-Latin`、`NFD` 等 |
| `dir` | string | 否 | 转换方向：`forward`（默认）或 `reverse` |

### 常用转换 ID

| ID | 说明 |
|----|------|
| `Cyrillic-Latin` | 西里尔字母 → 拉丁字母 |
| `Han-Latin` | 汉字 → 拉丁字母（拼音） |
| `Katakana-Latin` | 片假名 → 拉丁字母 |
| `Arabic-Latin` | 阿拉伯字母 → 拉丁字母 |
| `Any-Latin` | 任意脚本 → 拉丁字母 |
| `NFD` / `NFC` | Unicode 归一化 |
| `Lower` / `Upper` / `Title` | 大小写转换 |

## 相关链接

- [ICU 排序过滤器]({{< relref "./icu-collation" >}})
- [ICU 分析器]({{< relref "../analyzers/icu-analyzer" >}})
- [文本分析]({{< relref "/docs/features/mapping-and-analysis/text-analysis/" >}})


