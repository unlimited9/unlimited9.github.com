# mongodb

## mapReduce

#### usage
```
db.runCommand(
               {
                 mapReduce: <collection>,
                 map: <function>,
                 reduce: <function>,
                 finalize: <function>,
                 out: <output>,
                 query: <document>,
                 sort: <document>,
                 limit: <number>,
                 scope: <document>,
                 jsMode: <boolean>,
                 verbose: <boolean>,
                 bypassDocumentValidation: <boolean>,
                 collation: <document>,
                 writeConcern: <document>
               }
             )
```

#### map-reduce examples

`orders collection sample data`  
```
db.orders.insertMany([
    { _id: 1, cust_id: "Ant O. Knee", ord_date: new Date("2020-03-01"), price: 25, items: [ { sku: "oranges", qty: 5, price: 2.5 }, { sku: "apples", qty: 5, price: 2.5 } ], status: "A" },
    { _id: 2, cust_id: "Ant O. Knee", ord_date: new Date("2020-03-08"), price: 70, items: [ { sku: "oranges", qty: 8, price: 2.5 }, { sku: "chocolates", qty: 5, price: 10 } ], status: "A" },
   { _id: 3, cust_id: "Busby Bee", ord_date: new Date("2020-03-08"), price: 50, items: [ { sku: "oranges", qty: 10, price: 2.5 }, { sku: "pears", qty: 10, price: 2.5 } ], status: "A" },
   { _id: 4, cust_id: "Busby Bee", ord_date: new Date("2020-03-18"), price: 25, items: [ { sku: "oranges", qty: 10, price: 2.5 } ], status: "A" },
   { _id: 5, cust_id: "Busby Bee", ord_date: new Date("2020-03-19"), price: 50, items: [ { sku: "chocolates", qty: 5, price: 10 } ], status: "A"},
   { _id: 6, cust_id: "Cam Elot", ord_date: new Date("2020-03-19"), price: 35, items: [ { sku: "carrots", qty: 10, price: 1.0 }, { sku: "apples", qty: 10, price: 2.5 } ], status: "A" },
   { _id: 7, cust_id: "Cam Elot", ord_date: new Date("2020-03-20"), price: 25, items: [ { sku: "oranges", qty: 10, price: 2.5 } ], status: "A" },
   { _id: 8, cust_id: "Don Quis", ord_date: new Date("2020-03-20"), price: 75, items: [ { sku: "chocolates", qty: 5, price: 10 }, { sku: "apples", qty: 10, price: 2.5 } ], status: "A" },
   { _id: 9, cust_id: "Don Quis", ord_date: new Date("2020-03-20"), price: 55, items: [ { sku: "carrots", qty: 5, price: 1.0 }, { sku: "apples", qty: 10, price: 2.5 }, { sku: "oranges", qty: 10, price: 2.5 } ], status: "A" },
   { _id: 10, cust_id: "Don Quis", ord_date: new Date("2020-03-23"), price: 25, items: [ { sku: "oranges", qty: 10, price: 2.5 } ], status: "A" }
]);
```

`define map function`  
```
var _map_orders_price = function() {
   emit(this.cust_id, this.price);
};
```

`define reduce functions`  
```
var _reduce_sum = function(_key, _value) {
   return Array.sum(_value);
};
var _reduce_avg = function(_key, _value) {
   return Array.avg(_value);
};
```

`perform map-reduce on all documents in the orders`  
```
db.orders.mapReduce(
   _map_orders_price,
   _reduce_sum,
   { out: "orders_price_sum" }
);
db.orders.mapReduce(
   _map_orders_price,
   _reduce_avg,
   { out: "orders_price_avg" }
);


db.orders.mapReduce(
    function() {
        emit(this.cust_id, this.price);
    },
    function(_key, _value) {
        return Array.avg(_value);
    },
    {
        out: "orders_price_avg"
    }
);
```

#### aggregation alternative
```
db.orders.aggregate([
   { $group: { _id: "$cust_id", value: { $sum: "$price" } } },
   { $out: "orders_price_sum" }
]);
```


## java example
```
package com.journaldev.mongodb;

import java.net.UnknownHostException;

import com.mongodb.DB;
import com.mongodb.DBCollection;
import com.mongodb.DBObject;
import com.mongodb.MapReduceCommand;
import com.mongodb.MapReduceOutput;
import com.mongodb.MongoClient;

public class MongoDBMapReduce {

	public static void main(String[] args) throws UnknownHostException {

		// create an instance of client and establish the connection
		MongoClient m1 = new MongoClient();

		// get the test db,use your own
		DB db = m1.getDB("journaldev");

		// get the car collection
		DBCollection coll = db.getCollection("car");

		// map function to categorize overspeed cars
		String carMap = "function (){" + "var criteria;"
				+ "if ( this.speed > 70 ) {" + "criteria = 'overspeed';"
				+ "emit(criteria,this.speed);" + "}" + "};";

		// reduce function to add all the speed and calculate the average speed

		String carReduce = "function(key, speed) {" + "var total =0;"
				+ "for (var i = 0; i < speed.length; i++) {"
				+ "total = total+speed[i];" + "}"
				+ "return total/speed.length;" + "};";

		// create the mapreduce command by calling map and reduce functions
		MapReduceCommand mapcmd = new MapReduceCommand(coll, carMap, carReduce,
				null, MapReduceCommand.OutputType.INLINE, null);

		// invoke the mapreduce command
		MapReduceOutput cars = coll.mapReduce(mapcmd);

		// print the average speed of cars
		for (DBObject o : cars.results()) {

			System.out.println(o.toString());

		}

	}

}
```

## 9. Appendix

#### reference site

* Reference > Database Commands > Aggregation Commands > mapReduce > mapReduce  
https://docs.mongodb.com/manual/reference/command/mapReduce/


