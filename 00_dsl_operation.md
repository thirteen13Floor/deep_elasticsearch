# 1、创建索引
```
PUT scoring
{
  "settings":{
      "number_of_shards":1,
      "number_of_replicas":0
    }
}

POST scoring/doc/1
{
  "name":"first document"
}

POST scoring/_search
{
  "query":{
    "match":{"name":"document"}
  }
}

POST scoring/_search
{
  "explain": true, 
  "query":{
    "match":{"name":"document"}
  }
}

POST scoring/doc/2
{
  "name":"second example document"
}



PUT clients
POST clients/client/1
{
  "id":1,
  "name":"Joe"
}
POST clients/client/2
{
  "id":2,
  "name":"Jane"
}
POST clients/client/3
{
  "id":3,
  "name":"Jack"
}

POST clients/client/4
{
  "id":4,
  "name":"Rob"
}

POST clients/_search
{
"query":{
"prefix":{
  "name":"j"
  }
  }
}
```

# 2、 nested类型索引创建
```
PUT library
POST library/_mapping/book
{
  "book":{
    "properties":{
      "review":{
        "type":"nested",
        "properties":{
          "nickname":{"type":"text",
          "fields":{
          "keyword":{
            "type":"keyword"
          }
        }},
        "text":{"type":"text",
        "fields":{
          "keyword":{
            "type":"keyword"
          }
        }
        },
        "stars":{"type":"integer"
      }
    }
  }
}
}
}
```
#  3、批量导入数据
```
POST library/book/_bulk
{ "index": {"_index": "library", "_type": "book", "_id": "1"}}
{ "title": "All Quiet on the Western Front","otitle": "Im Westen nichts Neues","author": "Erich Maria Remarque","year": 1929,"characters": ["Paul Bumer", "Albert Kropp", "Haie Westhus", "Fredrich Mller", "Stanislaus Katczinsky", "Tjaden"],"tags": ["novel"],"copies": 1, "available": true, "section" : 3}
{ "index": {"_index": "library", "_type": "book", "_id": "2"}}
{ "title": "Catch-22","author": "Joseph Heller","year": 1961,"characters": ["John Yossarian", "Captain Aardvark", "Chaplain Tappman", "Colonel Cathcart", "Doctor Daneeka"],"tags": ["novel"],"copies": 6, "available" : false, "section" : 1}
{ "index": {"_index": "library", "_type": "book", "_id": "3"}}
{ "title": "The Complete Sherlock Holmes","author": "Arthur Conan Doyle","year": 1936,"characters": ["Sherlock Holmes","Dr. Watson", "G. Lestrade"],"tags": [],"copies": 0, "available" : false, "section" : 12}
{ "index": {"_index": "library", "_type": "book", "_id": "4"}}
{ "title": "Crime and Punishment","otitle": "pec ssss","author": "Fyodor Dostoevsky","year": 1886,"characters": ["Raskolnikov", "Sofia Semyonovna Marmeladova"],"tags": [],"copies": 0, "available" : true}
```

```
POST library/book/5
{
  "title":"The Sorows of Young Werther",
  "author":"Johann Wolfgang won Goethe",
  "avaliable":true,
  "characters":["Werther","Lotte","Albert","Fraulein von B"],
  "copies":1,
  "otitle":"Die Leiden des jungen Werthers",
  "section":4,
  "tags":["novel", "classics"],
  "year":1774,
  "review":[
    {
      "nickname":"Anna",
      "text":"could be good, but not my style",
      "stars":15
    }]
}

POST library/book/6
{
  "title":"The Peasants",
  "author":"Wdyslaw Reymojnt",
  "avaliable":true,
  "characters":["Boryna", "Jankiel","Jagna Paczesiona"],
  "copies":4,
  "otitle":"chopi",
  "section":4,
  "tags":["novel", "polish","classics"],
  "year":1904,
  "review":[
    {
      "nickname":"anonymous",
      "text":"awsome book",
      "stars":3
    },
    {
      "nickname":"Jane",
      "text":"Great book, but too long",
      "stars":4
    },
    {
      "nickname":"Rick",
      "text":"Why bother, when you can find it on the internet",
      "stars":32
    }]
}
```

# 4、全量检索
```
GET library/_search
{
  "query":{
    "match_all": {}
  }
}
```

# 5、范围检索
```
POST library/_search
{
  "query":{
    "range":{
      "copies": {
        "gte": 1,
        "lte": 3
      }
    }
  }
}
```

# 6、精确多值匹配
```
POST library/_search
{
  "query":{
    "terms": {
      "tags": ["novel", "polish", "classics", "criminal", "new"]
    }
  }
}
```

# 7、bool查询
```
GET library/_search
{
  "query":{
    "bool":{
      "must":[
        {
          "range":{
            "copies":{"gte" : 1}
          }
        }],
        "should": [
          {"range":{
            "year":{
              "gt":1950
            }
          }}
        ]
    }
  }
}
```

# 8、dis_max 检索

——dismax的核心就是解决：1个字段包含两个关键词比2个字段分别包含一个关键词得分要高的问题。
```
GET library/_search
{
  "_source":{
    "includes": [ "_id","_score"]
  },
  "query":{
    "dis_max": {
      "tie_breaker": 0.0,
      "boost": 1.0,
      "queries": [
        {"match":{"title":"crime punishment"}},
      {"match":{"characters":"raskolnikov"}}]
    }
  }
}
```

# 9、全文检索
```
GET library/_search
{
  "explain": true, 
  "_source":{
    "includes": [ "_id","_score"]
  },
  "query":
        {"match":{"title":"crime punishment"}}
    }
    
GET library/_mapping
GET library/_search
{
  "_source": {
    "includes": "tags"
  }, 
  "query":{
    "term":{
      "tags.keyword":"novel"
    }
  }
}

GET library/_search
{
  "query": {
    "common":{
      "title":{
        "query":"the western front",
        "cutoff_frequency":0.1,
        "low_freq_operator":"and"
      }
    }
  }
}

GET library/_search
{
  "query": {
   "bool":{
     "must": [
       {"term":{"title":"western"}},
       {"term":{"title":"front"}}
     ],
     "should": [
       {"term":{"title":"the"}}
     ]
   }
}
}

GET library/_search
{
  "query": {
    "match_all": {}
  }
}
```

# 10、原生检索方式——用途非常广和深远
```
GET library/_search
{
  "query": {
    "query_string": {
      "query": "+title:Sorows + title:Young +author:\"won Goethe\" - copies:{5 TO *]"
    }
  }
}

GET library/_search
{
  "query": {
    "query_string": {
      "default_field":"title", 
      "query": "Sorows +Young\""
    }
  }
}

GET library/_search
{
  "query": {
    "simple_query_string": {
      "fields":["title"], 
      "query": "Sorows +Young\""
    }
  }
}
```

# 11、前缀检索
```
GET library/_search
{
  "query": {
    "prefix": {
      "title": {
        "value": "wes"
      }
    }
  }
}

GET library/_search
{
  "query": {
    "regexp": {
      "characters": "wat..n"
    }
  }
}

GET library/_search
{
  "query": {
    "fuzzy": {
      "title": {
        "value": "crimea",
        "fuzziness":3,
        "max_expansions":50
      }
    }
  }
}
```
# 12、自定义评分检索
```
GET library/_search
{
  "query":{
    "function_score": {
      "query": {"match_all": {}},
      "score_mode": "multiply",
      "functions": [
        {"gauss":{
          "year": {
            "origin":2014,
            "scale": 2014,
            "offset": 0,
            "decay": 0.5
          }
        }}
      ]
    }
  }
}
```

# 13、给特定字段加权，以提升或减低打分，用来改变返回排序
```
GET library/_search
{
  "query": {
    "boosting": {
      "positive": {
        "match_all": {}
    },
    "negative_boost":{
      "term":{
        "available":false
      }
    }
  }
  }
}

GET library/_search
{
    "query": {
        "boosting" : {
            "positive" : {
                "match_all": {}
            },
            "negative" : {
                 "term":{
        "available":false
      }
            },
            "negative_boost" : 0.2
        }
    }
}

GET library/_search
{
  "query": {
    "match_phrase": {
      "otitle": "leiden des jungen"
    }
  }
}
```

# 14、位置敏感检索（很少用，不过社区有人提问过）
```
GET library/_search
{
  "query": {
    "span_near":{
      "clauses":[
        {"span_near": {
          "clauses": [
            {"span_term":{
              "otitle": "die"
            }},
            {"span_near": {
              "clauses": [
                {"span_term": {
                  "otitle": "des"
                }
                },
                {
                  "span_term": {
                    "otitle": "jungen"
                  }
                }
              ],
              "slop":0,
              "in_order": true
            }
            }
          ],
          "slop": 2,
          "in_order": false
        }
        },
        {
          "span_term": {
            "otitle": "werthers"
          }
        }
        ],
        "slop": 0,
        "in_order": true
    }
  }
}

GET  library/_mapping
POST library/_search
{
  "query":{
    "match_all":{}
  }
}
```

# 15、nested特定类型检索
```
GET library/_search
{
  "query": {
    "nested": {
      "path": "review",
      "query": {
        "range": {
          "review.stars": {
            "gte": 3
          }
        }
      }
    }
  }
}


GET library/_search
{
  "query": {
    "nested": {
      "path": "review",
      "score_mode": "max",
      "query": {
        "function_score": {
            "query":{"match_all": {}},
           "score_mode":"max",
           "boost_mode":"replace",
           "field_value_factor":{
             "field":"review.stars",
             "factor":1,
             "modifier":"none"
           }
            }
          }
        }
      }
    }
```
