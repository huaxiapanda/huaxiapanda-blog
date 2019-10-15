---
title: elasticsearch入门_01
permalink: elasticsearch入门_01
date: 2019-10-15 21:55:49
categories:
  - 大数据
  -  es
tags: elasticsearch
---
@[toc]
# 1 Elasticsearch聚合分析
## 1.1 需求1_计算每个tag下的商品数量
案例:进行聚合分析需要fielddata属性设置为true
```bash
{
  "properties": {
    "tags":{
      "type":"text",
      "fielddata": true
    }
  }
}
```
```bash
GET /ecommerce/product/_search
{
  "size": 0,
  "aggs": {
    "group_by_tags": {
      "terms": {
        "field": "tags"
      }
    }
  }
}
```
## 1.2 需求2_对名称中包含yagao的商品，计算每个tag下的商品数量
```bash
GET /ecommerce/product/_search
{
  "size": 0,
  "query": {
    "match": {
      "name": "yagao"
    }
  },
  "aggs": {
    "group_by_tags": {
      "terms": {
        "field": "tags"
      }
    }
  }
}
```
## 1.3 需求3_先分组，在计算每组的平均值，计算每个tag下的商品的平均价格
```bash
GET /ecommerce/product/_search
{
  "size": 0,
  "aggs": {
    "group_by_tags": {
      "terms": {
        "field": "tags"
      },
      "aggs": {
        "avg_price": {
          "avg": {
            "field": "price"
          }
        }
      }
    }
  }
}
```
## 1.4 需求4_计算每个tag下的商品的平均价格，并且按照平均价格降序排序
```bash
GET /ecommerce/product/_search
{
  "size": 0,
  "aggs": {
    "group_by_tags": {
      "terms": {
        "field": "tags",
        "order": {
          "avg_price": "desc"
        }
      },
      "aggs": {
        "avg_price": {
          "avg": {
            "field": "price"
          }
        }
      }
    }
  }
}
```
## 1.5 需求5_按照指定的价格范围区间进行分组，然后在每组内按照tag进行分组，最后在计算每组的平均价格
```bash
GET /ecommerce/product/_search
{
  "size": 0,
  "aggs": {
    "group_by_price": {
      "range": {
        "field": "price",
        "ranges": [
          {
            "from": 0,
            "to": 20
          },
          {
            "from": 20
            , "to": 40
          },
          {
            "from": 40,
            "to": 50
          }
        ]
      },
      "aggs": {
        "group_by_tags": {
          "terms": {
            "field": "tags"
          },
          "aggs": {
            "avg_price": {
              "avg": {
                "field": "price"
              }
            }
          }
        }
      }
    }
  }
}
```
