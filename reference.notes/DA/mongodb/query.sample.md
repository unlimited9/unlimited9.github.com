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
