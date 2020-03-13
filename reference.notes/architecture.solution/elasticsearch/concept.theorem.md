# concept

>Created 목요일 30 11월 2017
concept theorem

## 1. construction

## Cluster

### Node

#### master-eligible node
A node that has node.master set to true (default), which makes it eligible to be elected as the master node, which controls the cluster.

The master node is responsible for lightweight cluster-wide actions such as creating or deleting an index, tracking which nodes are part of the cluster, and deciding which shards to allocate to which nodes. It is important for cluster health to have a stable master node.

$ vi elasticsearch.yml
```
node.master: true
node.data: false
node.ingest: false
cluster.remote.connect: false
```

#### data node
A node that has node.data set to true (default). Data nodes hold data and perform data related operations such as CRUD, search, and aggregations.

Data nodes hold the shards that contain the documents you have indexed. Data nodes handle data related operations like CRUD, search, and aggregations. These operations are I/O-, memory-, and CPU-intensive. It is important to monitor these resources and to add more data nodes if they are overloaded.

$ vi elasticsearch.yml
```
node.master: false
node.data: true
node.ingest: false
cluster.remote.connect: false
```

#### ingest node
A node that has node.ingest set to true (default). Ingest nodes are able to apply an ingest pipeline to a document in order to transform and enrich the document before indexing. With a heavy ingest load, it makes sense to use dedicated ingest nodes and to mark the master and data nodes as node.ingest: false.

Ingest nodes can execute pre-processing pipelines, composed of one or more ingest processors. Depending on the type of operations performed by the ingest processors and the required resources, it may make sense to have dedicated ingest nodes, that will only perform this specific task.

$ vi elasticsearch.yml
```
node.master: false
node.data: false
node.ingest: true
cluster.remote.connect: false
```

#### coordinating node
Requests like search requests or bulk-indexing requests may involve data held on different data nodes. A search request, for example, is executed in two phases which are coordinated by the node which receives the client request — the coordinating node.

In the scatter phase, the coordinating node forwards the request to the data nodes which hold the data. Each data node executes the request locally and returns its results to the coordinating node. In the gather phase, the coordinating node reduces each data node’s results into a single global resultset.

Every node is implicitly a coordinating node. This means that a node that has all three node.master, node.data and node.ingest set to false will only act as a coordinating node, which cannot be disabled. As a result, such a node needs to have enough memory and CPU in order to deal with the gather phase.

If you take away the ability to be able to handle master duties, to hold data, and pre-process documents, then you are left with a coordinating node that can only route requests, handle the search reduce phase, and distribute bulk indexing. Essentially, coordinating only nodes behave as smart load balancers.

Coordinating only nodes can benefit large clusters by offloading the coordinating node role from data and master-eligible nodes. They join the cluster and receive the full cluster state, like every other node, and they use the cluster state to route requests directly to the appropriate place(s).

$ vi elasticsearch.yml
```
node.master: false
node.data: false
node.ingest: false
cluster.remote.connect: false
```

#### tribe node : deprecated
The tribe node is deprecated in favour of Cross-cluster search and will be removed in Elasticsearch 7.0.

The cross cluster search feature allows any node to act as a federated client across multiple clusters. In contrast to the tribe node feature, a cross cluster search node won’t join the remote cluster, instead it connects to a remote cluster in a light fashion in order to execute federated search requests.




## 9. Appendix

### reference site

#### Elasticsearch Reference [7.1] » Modules » Node
https://www.elastic.co/guide/en/elasticsearch/reference/current/modules-node.html

#### [Elasticsearch] 클러스터(Cluste), 노드(Node)
https://m.blog.naver.com/indy9052/220942459559

#### Elasticsearch - 엘라스틱서치 노드의 종류 그리고 클러스터링  
https://coding-start.tistory.com/182

#### Elasticsearch - 엘라스틱서치 자바 힙 메모리 변경(JVM Heap)  
https://coding-start.tistory.com/181?category=757916
