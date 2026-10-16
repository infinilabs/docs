---
title: "Painless 常用示例"
date: 0001-01-01
summary: "Painless 常用示例 #  本页收集了各种场景下常用的 Painless 脚本示例，可直接复制使用或按需修改。
脚本字段（Script Fields） #  在搜索结果中返回动态计算的字段值。
计算折扣价格 #  GET products/_search { &#34;query&#34;: { &#34;match_all&#34;: {} }, &#34;script_fields&#34;: { &#34;discounted_price&#34;: { &#34;script&#34;: { &#34;source&#34;: &#34;doc[&#39;price&#39;].value * (1 - params.discount)&#34;, &#34;params&#34;: { &#34;discount&#34;: 0.2 } } } } } 拼接姓名字段 #  GET employees/_search { &#34;script_fields&#34;: { &#34;full_name&#34;: { &#34;script&#34;: { &#34;source&#34;: &#34;&#34;&#34; def first = doc[&#39;first_name.keyword&#39;].size() &gt; 0 ? doc[&#39;first_name.keyword&#39;].value : &#39;&#39;; def last = doc[&#39;last_name."
---


# Painless 常用示例

本页收集了各种场景下常用的 Painless 脚本示例，可直接复制使用或按需修改。

## 脚本字段（Script Fields）

在搜索结果中返回动态计算的字段值。

### 计算折扣价格

```json
GET products/_search
{
  "query": { "match_all": {} },
  "script_fields": {
    "discounted_price": {
      "script": {
        "source": "doc['price'].value * (1 - params.discount)",
        "params": {
          "discount": 0.2
        }
      }
    }
  }
}
```

### 拼接姓名字段

```json
GET employees/_search
{
  "script_fields": {
    "full_name": {
      "script": {
        "source": """
          def first = doc['first_name.keyword'].size() > 0 ? doc['first_name.keyword'].value : '';
          def last = doc['last_name.keyword'].size() > 0 ? doc['last_name.keyword'].value : '';
          return last + first;
        """
      }
    }
  }
}
```

### 计算年龄

```json
GET users/_search
{
  "script_fields": {
    "age": {
      "script": {
        "source": """
          if (doc['birthday'].size() == 0) return null;
          ZonedDateTime birthday = doc['birthday'].value;
          ZonedDateTime now = ZonedDateTime.ofInstant(
            Instant.ofEpochMilli(System.currentTimeMillis()),
            ZoneId.of('UTC')
          );
          return ChronoUnit.YEARS.between(birthday, now);
        """
      }
    }
  }
}
```

## 文档更新

### 计数器递增

```json
POST page_views/_update/homepage
{
  "script": {
    "source": "ctx._source.view_count += params.count",
    "params": { "count": 1 }
  },
  "upsert": {
    "view_count": 1
  }
}
```

### 向数组追加唯一元素

```json
POST articles/_update/1
{
  "script": {
    "source": """
      if (ctx._source.tags == null) {
        ctx._source.tags = [];
      }
      for (tag in params.new_tags) {
        if (!ctx._source.tags.contains(tag)) {
          ctx._source.tags.add(tag);
        }
      }
    """,
    "params": {
      "new_tags": ["featured", "trending"]
    }
  }
}
```

### 从数组移除元素

```json
POST articles/_update/1
{
  "script": {
    "source": """
      if (ctx._source.tags != null) {
        ctx._source.tags.removeIf(tag -> tag == params.remove_tag);
      }
    """,
    "params": {
      "remove_tag": "outdated"
    }
  }
}
```

### 条件删除文档

```json
POST orders/_update/123
{
  "script": {
    "source": """
      if (ctx._source.status == 'cancelled' && ctx._source.refunded == true) {
        ctx.op = 'delete';
      } else {
        ctx.op = 'noop';
      }
    """
  }
}
```

### 嵌套对象更新

```json
POST users/_update/1
{
  "script": {
    "source": """
      if (ctx._source.profile == null) {
        ctx._source.profile = [:];
      }
      ctx._source.profile.bio = params.bio;
      ctx._source.profile.updated_at = params.now;
    """,
    "params": {
      "bio": "Easysearch 用户",
      "now": "2025-06-15T10:30:00Z"
    }
  }
}
```

## 批量更新（Update By Query）

### 为所有文档添加新字段

```json
POST my_index/_update_by_query
{
  "script": {
    "source": "ctx._source.version = params.version",
    "params": { "version": 2 }
  }
}
```

### 根据条件批量分类

```json
POST products/_update_by_query
{
  "script": {
    "source": """
      if (ctx._source.price < 100) {
        ctx._source.tier = 'budget';
      } else if (ctx._source.price < 500) {
        ctx._source.tier = 'mid-range';
      } else {
        ctx._source.tier = 'premium';
      }
    """
  }
}
```

### 批量格式转换

```json
POST logs/_update_by_query
{
  "script": {
    "source": """
      if (ctx._source.ip != null) {
        ctx._source.ip_parts = ctx._source.ip.splitOnToken('.');
      }
    """
  },
  "query": {
    "exists": { "field": "ip" }
  }
}
```

## 自定义评分

### 热度加权评分

```json
GET articles/_search
{
  "query": {
    "script_score": {
      "query": { "match": { "title": "技术" } },
      "script": {
        "source": """
          double textScore = _score;
          double popularity = Math.log(2 + doc['views'].value);
          double recency = 1.0;

          if (doc['publish_date'].size() > 0) {
            long ageInDays = (System.currentTimeMillis() - doc['publish_date'].value.toInstant().toEpochMilli()) / 86400000L;
            recency = 1.0 / (1 + ageInDays * 0.01);
          }

          return textScore * popularity * recency;
        """
      }
    }
  }
}
```

### 地理位置加权

```json
GET shops/_search
{
  "query": {
    "script_score": {
      "query": { "match_all": {} },
      "script": {
        "source": """
          double distance = doc['location'].arcDistance(params.lat, params.lon) / 1000.0;
          return 1.0 / (1 + distance);
        """,
        "params": {
          "lat": 39.9042,
          "lon": 116.4074
        }
      }
    }
  }
}
```

## 自定义排序

### 按优先级 + 时间综合排序

```json
GET tasks/_search
{
  "sort": [
    {
      "_script": {
        "type": "number",
        "script": {
          "source": """
            def priorityScore = 0;
            switch (doc['priority.keyword'].value) {
              case 'critical': priorityScore = 1000; break;
              case 'high': priorityScore = 100; break;
              case 'medium': priorityScore = 10; break;
              default: priorityScore = 1;
            }
            long ageHours = (System.currentTimeMillis() - doc['created_at'].value.toInstant().toEpochMilli()) / 3600000L;
            return priorityScore + ageHours;
          """
        },
        "order": "desc"
      }
    }
  ]
}
```

## 摄入管道脚本

### 自动生成摘要

```json
PUT _ingest/pipeline/auto_summary
{
  "processors": [
    {
      "script": {
        "source": """
          if (ctx.content != null && ctx.content.length() > 200) {
            ctx.summary = ctx.content.substring(0, 200) + '...';
          } else {
            ctx.summary = ctx.content;
          }
        """
      }
    }
  ]
}
```

### IP 地址分类

```json
PUT _ingest/pipeline/ip_classify
{
  "processors": [
    {
      "script": {
        "source": """
          if (ctx.client_ip != null) {
            if (ctx.client_ip.startsWith('10.') || ctx.client_ip.startsWith('192.168.')) {
              ctx.network_type = 'internal';
            } else {
              ctx.network_type = 'external';
            }
          }
        """
      }
    }
  ]
}
```

### 时间戳处理

```json
PUT _ingest/pipeline/timestamp_process
{
  "processors": [
    {
      "script": {
        "source": """
          if (ctx.timestamp != null) {
            def sdf = new SimpleDateFormat("yyyy-MM-dd'T'HH:mm:ss");
            def date = sdf.parse(ctx.timestamp);
            def cal = Calendar.getInstance();
            cal.setTime(date);
            ctx.hour_of_day = cal.get(Calendar.HOUR_OF_DAY);
            ctx.day_of_week = cal.get(Calendar.DAY_OF_WEEK);
            ctx.is_weekend = (ctx.day_of_week == 1 || ctx.day_of_week == 7);
          }
        """
      }
    }
  ]
}
```

## Reindex 中使用脚本

### 重建索引时重命名字段

```json
POST _reindex
{
  "source": {
    "index": "old_index"
  },
  "dest": {
    "index": "new_index"
  },
  "script": {
    "source": """
      ctx._source.user_name = ctx._source.remove('username');
      ctx._source.email_address = ctx._source.remove('email');
      if (ctx._source.status == 'inactive') {
        ctx.op = 'noop';
      }
    """
  }
}
```

### 重建索引时转换数据格式

```json
POST _reindex
{
  "source": {
    "index": "raw_logs"
  },
  "dest": {
    "index": "processed_logs"
  },
  "script": {
    "source": """
      // 将字符串时间戳转为统一格式
      if (ctx._source.log_time != null) {
        ctx._source.log_time = ctx._source.log_time.replace(' ', 'T') + 'Z';
      }

      // 提取日志级别
      if (ctx._source.message != null) {
        def matcher = /^\[(\w+)\]/.matcher(ctx._source.message);
        if (matcher.find()) {
          ctx._source.log_level = matcher.group(1);
        }
      }
    """
  }
}
```

## 实用技巧

### 空值安全处理模式

```painless
// 方式 1：检查 size
def val = doc['field'].size() > 0 ? doc['field'].value : defaultValue;

// 方式 2：使用 containsKey（在 ctx._source 中）
def val = ctx._source.containsKey('field') ? ctx._source.field : defaultValue;

// 方式 3：Elvis 运算符（对 params）
def val = params.value ?: defaultValue;
```

### Debug 输出（仅调试用）

```painless
// 在 Painless Execute API 中使用 Debug.explain 查看变量值
Debug.explain(doc['field'].value);
```

> **提示**：`Debug.explain` 会抛出一个包含变量类型和值的异常，仅在调试时使用，不要用于生产脚本。

