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
