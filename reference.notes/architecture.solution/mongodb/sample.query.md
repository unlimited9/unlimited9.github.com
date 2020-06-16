# mongodb

## database/collection/document


use statistics;  

db;  

show dbs;  

db.ad.parameters.insert({"adType": "iCw", "device": "Mobile", "parameters":1, "created": new Date()});  
>없으면 자동으로 collection 생성  
>db.createCollection("ad.parameters");
>>db.createCollection("articles", {  
    capped: true,  
    autoIndex: true,  
    size: 6142800,  
    max: 10000  
});  

show collections;  

> db.ad.parameters.drop()  

db.ad.parameters.insert([  
    {"adType": "iCw", "device": "Mobile", "parameters":2, "created": new Date()},  
    {"adType": "iCw", "device": "Pc", "parameters":3, "created": new Date()}  
]);  

db.ad.parameters.remove({"adType": "iCw", "device": "Mobile"})  

## document:find
db.ad.parameters.find();  
db.articles.find().pretty();  

#### Comparison
operator	설명  
$eq	(equals) 주어진 값과 일치하는 값  
$gt	(greater than) 주어진 값보다 큰 값  
$gte	(greather than or equals) 주어진 값보다 크거나 같은 값  
$lt	(less than) 주어진 값보다 작은 값  
$lte	(less than or equals) 주어진 값보다 작거나 같은 값  
$ne	(not equal) 주어진 값과 일치하지 않는 값  
$in	주어진 배열 안에 속하는 값  
$nin	주어빈 배열 안에 속하지 않는 값  

#### Logical
operator	설명  
$or	주어진 조건중 하나라도 true 일 때 true  
$and	주어진 모든 조건이 true 일 때 true  
$not	주어진 조건이 false 일 때 true  
$nor	주어진 모든 조건이 false 일때 true  

db.ad.parameters.find({"parameters": {$gt: 10, $lt: 30}});  
db.ad.parameters.find({"adType": {$in: [“iCw”, “iHu”]}});  
db.ad.parameters.find({$and: [{"adType": "iCw"}, {"device": "PC"}]})  

db.ad.parameters.find({"adType": /^iC[a-z]/})  
db.ad.parameters.find({$where: "this.adType == 'iCw'" })  


#### projection
db.ad.parameters.find({}, {"_id": false, "adType": true, "device": true, "parameters": true, "created": true, "lastModified": true});  
>db.getCollection('ad.parameters').aggregate([
    {$match:{}},
    {$project:{"_id": false, "adType": true, "device": true, "parameters": true, "created": {$dateToString: {format: "%Y-%m-%d", date: "$created"}}, "lastModified": {$dateToString: {format: "%Y-%m-%d", date: "$lastModified"}}}}
]);

db.ad.parameters.find({}).sort({"_id": 1}) //asc: 1, desc: -1  
db.ad.parameters.find({}).limit(1)  
db.ad.parameters.find({}).skip(1)  

var showPage = function(document, pageIdx, recordSize){  
    return document.find({}, {"_id": false, "adType": true, "device": true, "parameters": true}).sort({"_id": 1}).skip((pageIdx-1)*recordSize).limit(recordSize);  
};  
showPage(db.ad.parameters, 1, 10);

## document:update
`field`  
db.ad.parameters.update({"adType": "iCw", "device": "Mobile"}, {$set: {"parameters": 20}, $currentDate: {lastModified: true}})  
db.ad.parameters.update({"adType": "iCw", "device": "Mobile"}, {$inc: {"parameters": 1}, $currentDate: {lastModified: true}})  
db.ad.parameters.update({"adType": "iCw", "device": "Mobile"}, {$unset: {"parameters": 1}, $currentDate: {lastModified: true}})

`document with upsert/multi`  
db.ad.parameters.update({"adType": "iCw", "device": "Mobile"}, {$set: {"adType": "iCw", "device": "Mobile", "parameters": 1}, $currentDate: {lastModified: true}}, {upsert: true, multi: true})


`push/pull`  
>add sample data  
db.ad.parameters.update({"adType": "iCw", "device": "TV"}, {"adType": "iCw", "device": "TV", "parameters": 1, "created": new Date(), "manufacturer":[{"name": "samsung", "value": 1}, {"name": "lg": "value": 1}]}, { upsert: true })

db.ad.parameters.update({"adType": "iCw", "device": "TV", "modifyDate": new Date()}, {$push: {"manufacturer": {"name": "apple", "value": 1}}}, {multi: true})
db.ad.parameters.update({"adType": "iCw", "device": "TV", "modifyDate": new Date()}, {$pull: {"manufacturer": {"name": "apple"}}})

`multi push/pull`  
db.ad.parameters.update({"adType": "iCw", "device": "TV", "modifyDate": new Date()}, {
    $push: {
        "manufacturer": {
            $each: [{"name": "hwawei", "value": 1}, {"name": "nokia", "value": 1}],
            $sort: -1
        }
    }
}, {multi: true})

db.ad.parameters.update({"adType": "iCw", "device": "TV", "modifyDate": new Date()}, {
    $pull: {
        "manufacturer": {"name": {$in: ["hwawei", "nokia"]}},
        $sort: -1
    }
}, {multi: true})


## index

- 기본 인덱스 _id
- Single(단일) 필드 인덱스
- Compound (복합)  필드 인덱스
- Multikey 인덱스
- Geospatial(공간적) Index
- Text 인덱스
- Hashed 인덱스

db.ad.parameters.getIndexes()

#### create
db.ad.parameters.createIndex({adType: 1})
db.ad.parameters.find({adType: "iCw"})

db.ad.parameters.createIndex({adType: 1, device: -1})
db.ad.parameters.find({"adType": "iCw", "device": "Mobile"})

#### property
`Unique (유일함) 속성`  
db.ad.parameters.createIndex(  
    {adType: 1, device: -1},  
    { unique: true}  
)  

`Partial (부분적) 속성`  
db.ad.parameters.createIndex(  
    {adType: 1, device: -1},  
    {partialFilterExpression: {parameters: {$gt: 1000}}}  
)  

`TLL 속성`  
db.ad.parameters.createIndex(  
    {"modifyDate": 1},  
    {expireAfterSeconds: 3600}  
)  
>remove expired document: 1 / 60sec  

#### remove
db.ad.parameters.dropIndex({adType: 1})  
db.ad.parameters.dropIndexes() //_id index 제외  



#### find
```
db.getCollection('CIData_00').find({
//    $match: {
    //            $and : [
    //                {
                    'iUm.data.domain': /.*livingpick.com.*/
    //                }
    //            ]
//    }
});
```

#### aggregate
```
db.getCollection('CIData_00').aggregate(
    { 
        $match: {
    //            $and : [
    //                {
                    'iUm.data': { $exists : true }
    //                }
    //            ]
        }
    }, {
        $group: {
            _id : null,
            sum : {
                $sum: {
                    $size : '$iUm.data'
                }
             }
         }
    }
);
```

#### count
```
db.getCollection('CIData_00').count({'iUm.data.domain': /.*livingpick.com.*/})
db.getCollection('CIData_00').count({'iUm.data.domain': /.*dabagirl.co.kr.*/})
```


---

>> 광고타겟팅별
```
// adtype:iCw parameter
db.getCollection('CIData_00').count(
    {'iCw.data':{$exists:1}}
);
```

>> 광고타겟팅/캠페인코드별
```
// adtype:iCw:siteCode parameter
db.getCollection('CIData_00').aggregate([
    {$match:{'iCw.data':{$exists:1}}},
    {$project:{id:true, adType:'iCw', data:{$setUnion:'$iCw.data.siteCode'}}},
    {$unwind:'$data'},
    {$group: {_id: {adType:'$adType', siteCode:'$data'}, parameter: { $sum: 1 }}}
]);
```

>> 광고타겟팅 데이터에서 추출한 캠페인코드별
```
// adtype:iCw > siteCode parameter
db.getCollection('CIData_00').aggregate([
    {$match:{'iCw.data':{$exists:1}}},
    {$project:{id:true, adType:'iCw', data:{$setUnion:'$iCw.data.siteCode'}}},
    {$unwind:'$data'},
    {$group: {_id: {siteCode:'$data'}, parameter: { $sum: 1 }}}
]);

// device parameter
db.getCollection('CIData_00').aggregate([
    {$group: {_id: '$saveCnt.device', parameter: { $sum: 1 }}}
]);

db.getCollection('CIData_00').aggregate([
    {$match:{'iCw.data':{$exists:1}}},
    {$project:{id:true, adType:'iCw', device:'$saveCnt.device'}},
    {$group: {_id: {adType:'$adType', device:'$device'}, parameter: { $sum: 1 }}}
]);
```
---



>> 누적 TOTAL  
`상품(iCw,iSr)`  
```
db.getCollection('CIData_00').aggregate([
     {$project:{ 'iCw':{$cond:[{$and : [{$eq :[{$type:'$iCw.data'}, 'array']}, {$gte:[{$size:'$iCw.data'}, 1]}]},1,0]}
          ,'device':{$switch:{ branches:[ {case:{$eq:['$saveCnt.device', 'P']} , then:'PC'}, {case:{$eq:['$saveCnt.device', 'M']} , then:'MOBILE'} ], default:'UNKNOWN'} }}}
    ,{$group:{'iCw':{$sum:'$iCw'}, '_id':'$device', 'total':{$sum:1}}}
]);
```

`도메인(iUm)`  
```
db.getCollection('CIData_00').aggregate([
     {$project:{ 'iUm':{$cond:[{$and : [{$eq :[{$type:'$iUm.data'}, 'array']}, {$gte:[{$size:'$iUm.data'}, 1]}]},1,0]}
          ,'device':{$switch:{ branches:[ {case:{$eq:['$saveCnt.device', 'P']} , then:'PC'}, {case:{$eq:['$saveCnt.device', 'M']} , then:'MOBILE'} ], default:'UNKNOWN'} }}}
    ,{$group:{'iUm':{$sum:'$iUm'}, '_id':'$device', 'total':{$sum:1}}}
]);
```

`해비유저(iHu)`  
```
db.getCollection('CIData_00').aggregate([
     {$match:{'iHu.data.visitCnt':{$gte:3}}},
     {$project:{ 'iHu':{$cond:[{$and : [{$eq :[{$type:'$iHu.data'}, 'array']}, {$gte:[{$size:'$iHu.data'}, 1]}]},1,0]}
          ,'device':{$switch:{ branches:[ {case:{$eq:['$saveCnt.device', 'P']} , then:'PC'}, {case:{$eq:['$saveCnt.device', 'M']} , then:'MOBILE'} ], default:'UNKNOWN'} }}}
    ,{$group:{'iHu':{$sum:'$iHu'}, '_id':'$device', 'total':{$sum:1}}}
]);
```

>> 광고주별  
`상품(iCw,iSr)`  
```
db.getCollection('CIData_00').aggregate([
    {$match:{'iCw.data':{$exists:1}}},
    {$project:{_id:true, device:'$saveCnt.device', data:{$setUnion:'$iCw.data.siteCode'}}},
    {$unwind:'$data'},
    {$project:{targetData:'$data',
                'PC':{$cond:{if:{$eq:['$device', 'P']}, then:1, else:0}},
                'MOBILE':{$cond:{if:{$eq:['$device', 'M']}, then:1, else:0}},
                'UNKNOWN':{$cond:{if:{$eq:['$device', undefined]}, then:1, else:0}} }},
    {$group:{_id:'$targetData', 'PC':{$sum:'$PC'}, 'MOBILE':{$sum:'$MOBILE'}, 'UNKNOWN':{$sum:'$UNKNOWN'}, 'TOTAL':{$sum:1}}}
]);
```
