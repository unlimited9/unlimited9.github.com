# 02.administration : elasticsearch

>Created 월요일 20 11월 2017
elasticsearch index management and query

## management

### queries
```
#### search all index
GET _search
{
  "query": {
	"match_all": {}
  }
}

#### cluster status
GET _cat/health?v

#### node list
GET _cat/nodes?v

#### index list
GET _cat/indices?v

#### create index
PUT /product
{
  "settings" : {
    "number_of_shards": 1,
    "number_of_replicas": 0,
    "index":{
      "analysis":{
        "analyzer":{
          "korean":{
            "filter":[
              "npos_filter",
              "nori_readingform",
              "lowercase"
            ],
            "tokenizer":"nori_user_dict"
          }
        },
        "tokenizer": {
          "nori_user_dict": {
            "mode":"MIXED",
            "type": "nori_tokenizer",
						"user_words" : [],
            "user_dictionary": "userdic_ko.txt",
            "index_poses" : ["N","SL","SH","SN","XR","V","UNK"]
          }
        },
        "filter":{
          "npos_filter": {
            "type": "nori_part_of_speech",
            "stoptags":[
              "E","IC","J","MAG","MM","SP","SSC",
              "SSO","SC","SE","XPN","XSA",
              "XSN","XSV","UNA","NA","VSV"
            ]
          }
        }
      }
    }
  },
	"aliases": {
		"prod" : {}
	},
  "mappings": {
    "properties": {
      "code": {
        "type": "keyword"
      },
			"name": {
				"type": "text",
		        "fields": {
		          "raw": { 
		            "type":  "keyword"
		          },
				"analyzer": "korean"
			},
      "keywords": {
        "type": "keyword",
        "copy_to": "fullsearch"
      },
			"category": {
				"type": "text",
				"fields": {
					"keyword": {
						"type": "keyword",
						"ignore_above": 256
					}
				},
				"analyzer": "korean"
			},
      "description": {
        "type": "text",
        "analyzer": "korean",
        "copy_to": "fullsearch"
      },
      "producer": {
        "type": "text",
        "analyzer": "korean",
        "copy_to": "fullsearch"
      },
      "asid": {
        "type": "text"
      },
      "site": {
        "type": "text"
      },
      "url": {
        "type": "text"
      },
      "price": {
        "type": "float"
      },
      "currency": {
        "type": "keyword"
      },
      "discountRate": {
        "type": "float"
      },
      "status": {
        "type": "keyword"
      },
      "registerDate": {
        "type": "date",
				"format": "strict_date_optional_time||epoch_millis"
      },
      "modifyDate": {
        "type": "date",
				"format": "strict_date_optional_time||epoch_millis"
      },
      "fullsearch": {
        "type":"text"
      }
    }
  }
}

#### index info
GET /product

#### index(aliase) info
GET /prod

#### text analyze 
GET /prod/_analyze?pretty
{
	"analyzer" : "korean",
	"text" : "아버지가 방에 들어간다"
}

GET _analyze
{
  "analyzer": "nori",
  "text" : "아버지가 방에 들어가신다"
}
GET _analyze
{
  "analyzer": "nori",
  "text" : "아버지 가방에 들어가신다"
}


#### insert document
POST /product/_doc/1
{
  "code": "033001000183",
  "name": "샬롯 볼륨업 브라세트 [ra100]",
  "keywords": "속옷,브라",
  "category": "옷",
  "description": "",
  "producer": "",
  "asid": "planets",
  "site": "gumzzi.co.kr",
  "url": "http://www.gumzzi.co.kr/shop/shopdetail.html?branduid=88292&xcode=179&mcode=000&scode=&sort=order&cur_code=179",
  "price": "19800.00",
  "currency": "KRW",
  "discountRate": "0.00",
  "status": "active",
  "registerDate": null,
  "modifyDate": null
  
}

#### index search all
GET /prod/_search
{
  "query": {
	"match_all": {}
  }
}

#### search index query
GET /prod/_search
{
  "query": {
  	"match": {
  	  "name" : "볼륨 브라"
  	}
  }
}

GET /prod/_search
{
  "query": {
  	"bool": {
  	  "should": [
    		{
    		  "match": {
      			"name": {
      			  "query": "볼륨",
      			  "_name": "firstQuery"
      			}
    		  }
    		},
    		{
    		  "match": {
      			"name": {
      			  "query": "브라",
      			  "_name": "secondQuery"
      			}
    		  }
    		}
  	  ]
  	}
  }
}

GET /prod/_search
{
  "query": {
  	"bool": {
  	  "must": [
    		{
    		  "match": {
    			  "name": {
    			    "query": "볼륨",
    		  	  "_name": "firstQuery"
    	  		}
    		  }
    		},
    		{
    		  "match": {
      			"name": {
      			  "query": "브라",
      			  "_name": "secondQuery"
      			}
    		  }
    		}
  	  ]
  	}
  }
}

#### search index query and filter
GET /prod/_search
{
  "query": {
  	"bool": {
  	  "must": [
  		{
  		  "match": {
  			  "name": "볼륨"
  		  }
  		},
  		{
  		  "match": {
  			  "name": "브라"
  		  }
  		}
  	  ],
  	  "filter": {
    		"match": {
    		  "status": "active" 
    		}
  	  }
  	}
  }
}

GET /prod/_search
{
  "query": {
  	"bool": {
  	  "must": [
    		{
    		  "match": {
    			  "name": "브라"
    		  }
    		},
    		{
    		  "match": {
    			"category": "옷"
    		  }
    		}
  	  ],
  	  "filter": {
    		"match": {
    		  "status": "active" 
    		}
  	  }
  	}
  }
}

GET /_aliases

#### index aliase
POST /_aliases
{
  "actions": [
  	{
  	  "remove": {
  		"alias": "shopdata",
  		"index": "product"
  	  }
  	},
  	{
  	  "add": {
  		"alias": "prod",
  		"index": "product"
  	  }
  	}
	]
}

#### delete index
DELETE /product

GET /prod
```

