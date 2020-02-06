#### 상품 Index
```
PUT /product
{
  "settings" : {
    "number_of_shards": 3,
    "number_of_replicas": 3,
    "max_shingle_diff":3,
    "index":{
      "analysis":{
        "analyzer":{
          "korean":{
            "filter":[
              "npos_filter",
              "nori_readingform",
              "lowercase",
              "custom_shingle"
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
            "index_poses" : ["N","SL","SH","SN","XR","V","UNK"],
            "decompound_mode": "discard"
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
          },
          "custom_shingle": {
            "type": "shingle",
            "token_separator": "",
            "min_shingle_size" : 2,
            "max_shingle_size": 3
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
      "adverId": {
        "type": "text"
      },
      "productCode": {
        "type": "keyword"
      },
      "productName": {
        "type": "text",
        "analyzer": "korean",
        "search_analyzer": "standard"
      },
      "topCategory": {
        "type": "text",
        "analyzer": "korean",
        "search_analyzer": "standard"
      },
      "status": {
        "type": "keyword"
      },
      "liveStatus": {
        "type": "text"
      },
      "kakaoStatus": {
        "type": "text"
      },
      "nation": {
        "type": "text"
      },
      "lastUpdate": {
        "type" : "date",
        "format" : "yyyy-MM-dd HH:mm:ss"
      },
      "visitCount": {
        "type": "float"
      }
    }
  }
}

## 상품 수집 정책 Index
PUT /config
{
  "settings" : {
    "number_of_shards": 3,
    "number_of_replicas": 3,
    "index":{
    }
  },
  "aliases": {
    "conf" : {}
  },
  "mappings": {
    "properties": {
      "adverId": {
        "type": "text"
      },
      "nationCode": {
        "type": "text"
      },
      "epYN": {
        "type": "text"
      },
      "responsiveYn": {
        "type": "text"
      },
      "categoryGatheringPolicy": {
        "type": "text"
      },
      "categoryValue": {
        "type": "text"
      },
      "productNameGatheringPolicy": {
        "type": "text"
      },
      "productNamesValue": {
        "type": "text"
      },
      "filterPurlUseYN": {
        "type": "text"
      },
      "filterPurlValue": {
        "type": "text"
      },
      "replacePurlParameterUseYN": {
        "type": "text"
      },
      "replacePurlParameterValue": {
        "type": "text"
      },
      "replaceWebProductUrlUseYN": {
        "type": "text"
      },
      "replaceWebPurlValue": {
        "type": "text"
      },
      "replaceMobProductUrlUseYN": {
        "type": "text"
      },
      "replaceMobPurlValue": {
        "type": "text"
      },
      "filterImgUrlUseYN": {
        "type": "text"
      },
      "filterImgUrlValue": {
        "type": "text"
      },
      "replaceImgUrlUseYN": {
        "type": "text"
      },
      "replaceImgUrlValue": {
        "type": "text"
      },
      "zeroPriceGatheringYn": {
        "type": "text"
      }
    }
  }
}

```
