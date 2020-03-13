## Elasticsearch 기본 한글 형태소 분석기 노리 (nori)  
  
/apps/elasticsearch/7.1.1/bin/elasticsearch-plugin install analysis-nori

/apps/elasticsearch/7.1.1/bin/elasticsearch-plugin remove analysis-nori

## 사용자 정의사전 추가

사용자 정의사전을 못 찾아서 나는 에러
```
{
  "error": {
    "root_cause": [
      {
        "type": "remote_transport_exception",
        "reason": "[dmoespd02][192.168.104.52:6540][indices:admin/create]"
      }
    ],
    "type": "illegal_argument_exception",
    "reason": "IOException while reading user_dictionary: /apps/elasticsearch/7.1.1/config/userdic_ko.txt",
    "caused_by": {
      "type": "no_such_file_exception",
      "reason": "/apps/elasticsearch/7.1.1/config/userdic_ko.txt"
    }
  },
  "status": 400
}
```    

$ touch /apps/elasticsearch/7.1.1/config/userdic_ko.txt

