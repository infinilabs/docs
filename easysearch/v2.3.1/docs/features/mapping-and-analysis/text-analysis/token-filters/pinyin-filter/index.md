---
title: "拼音过滤器（Pinyin Filter）"
date: 0001-01-01
summary: "拼音过滤器 #  pinyin 词元过滤器将中文词元转换为拼音表示，来自 analysis-pinyin 插件。
前提条件 #  需要安装 analysis-pinyin 插件：
bin/easysearch-plugin install analysis-pinyin 功能说明 #  此过滤器可以：
 将汉字转换为拼音全拼或首字母 保留或移除原始中文词元 支持多种输出格式组合  使用示例 #  PUT my-pinyin-index { &#34;settings&#34;: { &#34;analysis&#34;: { &#34;filter&#34;: { &#34;my_pinyin&#34;: { &#34;type&#34;: &#34;pinyin&#34;, &#34;keep_full_pinyin&#34;: true, &#34;keep_first_letter&#34;: true, &#34;keep_original&#34;: true, &#34;limit_first_letter_length&#34;: 16 } }, &#34;analyzer&#34;: { &#34;pinyin_analyzer&#34;: { &#34;type&#34;: &#34;custom&#34;, &#34;tokenizer&#34;: &#34;ik_smart&#34;, &#34;filter&#34;: [&#34;my_pinyin&#34;] } } } } } 测试效果 #  GET /_analyze { &#34;tokenizer&#34;: &#34;ik_smart&#34;, &#34;filter&#34;: [{&#34;type&#34;: &#34;pinyin&#34;, &#34;keep_full_pinyin&#34;: true}], &#34;text&#34;: &#34;中华人民共和国&#34; } 参数 #     参数 默认值 说明     keep_first_letter true 保留拼音首字母（如 &ldquo;中国&rdquo; → zg）   keep_full_pinyin true 保留全拼（如 &ldquo;中&rdquo; → zhong）   keep_joined_full_pinyin false 连接全拼（如 &ldquo;中国&rdquo; → zhongguo）   keep_original false 保留原始中文词元   keep_none_chinese true 保留非中文字符   keep_none_chinese_in_first_letter true 首字母模式中保留非中文   keep_none_chinese_together true 非中文字符连续保留   none_chinese_pinyin_tokenize true 拼音字母也参与分词   limit_first_letter_length 16 首字母最大长度   lowercase true 拼音小写   trim_whitespace true 移除空白   remove_duplicated_term false 移除重复词项    相关链接 #    拼音分析器  拼音分词器  "
---


# 拼音过滤器

`pinyin` 词元过滤器将中文词元转换为拼音表示，来自 analysis-pinyin 插件。

## 前提条件

需要安装 analysis-pinyin 插件：

```bash
bin/easysearch-plugin install analysis-pinyin
```

## 功能说明

此过滤器可以：

- 将汉字转换为拼音全拼或首字母
- 保留或移除原始中文词元
- 支持多种输出格式组合

## 使用示例

```json
PUT my-pinyin-index
{
  "settings": {
    "analysis": {
      "filter": {
        "my_pinyin": {
          "type": "pinyin",
          "keep_full_pinyin": true,
          "keep_first_letter": true,
          "keep_original": true,
          "limit_first_letter_length": 16
        }
      },
      "analyzer": {
        "pinyin_analyzer": {
          "type": "custom",
          "tokenizer": "ik_smart",
          "filter": ["my_pinyin"]
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
  "tokenizer": "ik_smart",
  "filter": [{"type": "pinyin", "keep_full_pinyin": true}],
  "text": "中华人民共和国"
}
```

## 参数

| 参数 | 默认值 | 说明 |
|------|--------|------|
| `keep_first_letter` | `true` | 保留拼音首字母（如 "中国" → `zg`） |
| `keep_full_pinyin` | `true` | 保留全拼（如 "中" → `zhong`） |
| `keep_joined_full_pinyin` | `false` | 连接全拼（如 "中国" → `zhongguo`） |
| `keep_original` | `false` | 保留原始中文词元 |
| `keep_none_chinese` | `true` | 保留非中文字符 |
| `keep_none_chinese_in_first_letter` | `true` | 首字母模式中保留非中文 |
| `keep_none_chinese_together` | `true` | 非中文字符连续保留 |
| `none_chinese_pinyin_tokenize` | `true` | 拼音字母也参与分词 |
| `limit_first_letter_length` | `16` | 首字母最大长度 |
| `lowercase` | `true` | 拼音小写 |
| `trim_whitespace` | `true` | 移除空白 |
| `remove_duplicated_term` | `false` | 移除重复词项 |

## 相关链接

- [拼音分析器]({{< relref "../analyzers/pinyin-analyzer" >}})
- [拼音分词器]({{< relref "../tokenizers/pinyin" >}})


