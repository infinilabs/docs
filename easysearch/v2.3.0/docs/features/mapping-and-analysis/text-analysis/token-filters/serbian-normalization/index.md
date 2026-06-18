---
title: "塞尔维亚语归一化过滤器（Serbian Normalization）"
date: 0001-01-01
summary: "塞尔维亚语归一化过滤器 #  serbian_normalization 词元过滤器将塞尔维亚语西里尔字母转写为对应的拉丁字母形式，使两种书写系统的文本在搜索时可以互相匹配。
归一化规则 #  塞尔维亚语同时使用西里尔字母和拉丁字母，此过滤器将西里尔形式统一为拉丁形式：
   西里尔字母 拉丁字母 西里尔字母 拉丁字母     а a п p   б b р r   в v с s   г g т t   д d ћ ć   ђ đ у u   е e ф f   ж ž х h   з z ц c   и i ч č   ј j џ dž   к k ш š   л l љ lj   м m њ nj   н n     о o      使用示例 #  PUT my-serbian-index { &#34;settings&#34;: { &#34;analysis&#34;: { &#34;filter&#34;: { &#34;serbian_norm&#34;: { &#34;type&#34;: &#34;serbian_normalization&#34; } }, &#34;analyzer&#34;: { &#34;my_serbian&#34;: { &#34;type&#34;: &#34;custom&#34;, &#34;tokenizer&#34;: &#34;standard&#34;, &#34;filter&#34;: [&#34;lowercase&#34;, &#34;serbian_norm&#34;] } } } } } 测试效果 #  GET /_analyze { &#34;tokenizer&#34;: &#34;standard&#34;, &#34;filter&#34;: [&#34;serbian_normalization&#34;], &#34;text&#34;: &#34;Београд&#34; } 响应中 Београд（西里尔）→ Beograd（拉丁）。"
---


# 塞尔维亚语归一化过滤器

`serbian_normalization` 词元过滤器将塞尔维亚语西里尔字母转写为对应的拉丁字母形式，使两种书写系统的文本在搜索时可以互相匹配。

## 归一化规则

塞尔维亚语同时使用西里尔字母和拉丁字母，此过滤器将西里尔形式统一为拉丁形式：

| 西里尔字母 | 拉丁字母 | 西里尔字母 | 拉丁字母 |
|-----------|---------|-----------|---------|
| а | a | п | p |
| б | b | р | r |
| в | v | с | s |
| г | g | т | t |
| д | d | ћ | ć |
| ђ | đ | у | u |
| е | e | ф | f |
| ж | ž | х | h |
| з | z | ц | c |
| и | i | ч | č |
| ј | j | џ | dž |
| к | k | ш | š |
| л | l | љ | lj |
| м | m | њ | nj |
| н | n | | |
| о | o | | |

## 使用示例

```json
PUT my-serbian-index
{
  "settings": {
    "analysis": {
      "filter": {
        "serbian_norm": {
          "type": "serbian_normalization"
        }
      },
      "analyzer": {
        "my_serbian": {
          "type": "custom",
          "tokenizer": "standard",
          "filter": ["lowercase", "serbian_norm"]
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
  "filter": ["serbian_normalization"],
  "text": "Београд"
}
```

响应中 `Београд`（西里尔）→ `Beograd`（拉丁）。

## 参数

此过滤器不接受任何参数。

## 适用场景

- 索引中混合包含塞尔维亚语西里尔和拉丁文本
- 用户可能用任一书写系统搜索
- 无需区分同一单词的西里尔与拉丁拼写

