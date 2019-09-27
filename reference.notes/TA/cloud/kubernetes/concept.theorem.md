# Kubernetes(k8s) : concept and theorem

>Created 목요일 20 2월 2017
automating deployment, scaling, and management of containerized applications.


## 1. Introduce
Kubernetes is a portable, extensible, open-source platform for managing containerized workloads and services, that facilitates both declarative configuration and automation.

MSA(Micro Service Architecture) : 1~2CPU로 구성해 운영가능한 서비스 

## 2. Concept/Features

Kubernetes는 컨테이너 운영환경(Docker 등)을 관리하기 위한 플랫폼이다.
- Components : Master / Node
- Pod : 서비스 단위 배포 모듈
- Controller : Pod Cluster 구성/관리 (Replication Controller 등) => Deployment로 추상화
(Replication Controller (aka RC), Replication Set, DaemonSet, Job, StatefulSet, Deployment)
	- StatefulSet : Pod 이름에 대한 규칙성 부여, 배포시 순차적인 기동과 업데이트, 개별 Pod에 대한 디스크 볼륨 관리
- Service : 네트워크 분산 설정 - L4 레이어로 TCP 단에서 Pod들을 밸런싱
	- Headless Service : 
- Ingress : HTTP(S)기반의 L7 로드밸런싱 기능을 제공하는 컴포넌트


### Kubernetes Cluster Architecture
the architecture of a Kubernetes cluster without the cloud controller manager.![Pre CCM Kube Arch](https://d33wubrfki0l68.cloudfront.net/e298a92e2454520dddefc3b4df28ad68f9b91c6f/70d52/images/docs/pre-ccm-arch.png)

### Kubernetes Architecture with the CCM(Cloud Controller Manager)![CCM Kube Arch](https://d33wubrfki0l68.cloudfront.net/518e18713c865fe67a5f23fc64260806d72b38f5/61d75/images/docs/post-ccm-arch.png)





### Components

#### Master Components
Master components provide the cluster’s control plane. Master components make global decisions about the cluster (for example, scheduling), and they detect and respond to cluster events (for example, starting up a new pod when a deployment’s replicas field is unsatisfied).
- `kube-apiserver` : Component리 on the master that exposes the Kubernetes API. It is the front-end for the Kubernetes control plane.
- `etcd` : Consistent and highly-available key value store used as Kubernetes’ backing store for all cluster data.
- `kube-scheduler` : Component on the master that watches newly created pods that have no node assigned, and selects a node for them to run on.
- `kube-controller-manager` : Component on the master that runs controllers .
- `cloud-controller-manager` : cloud-controller-manager runs controllers that interact with the underlying cloud providers.

#### Node Components
Node components run on every node, maintaining running pods and providing the Kubernetes runtime environment.
- `kubelet` : An agent that runs on each node in the cluster. It makes sure that containers are running in a pod.
- `kube-proxy` : kube-proxy is a network proxy that runs on each node in the cluster.
- `Container Runtime` : Kubernetes supports several container runtimes: Docker, containerd, cri-o, rktlet and any implementation of the Kubernetes CRI (Container Runtime Interface).

#### Addons
Addons use Kubernetes resources (DaemonSet , Deployment , etc) to implement cluster features. Because these are providing cluster-level features, namespaced resources for addons belong within the kube-system namespace.
- `DNS` : Cluster DNS is a DNS server, in addition to the other DNS server(s) in your environment, which serves DNS records for Kubernetes services.
- `Web UI (Dashboard)` : Dashboard is a general purpose, web-based UI for Kubernetes clusters. It allows users to manage and troubleshoot applications running in the cluster, as well as the cluster itself.
- `Container Resource Monitoring` : Container Resource Monitoring records generic time-series metrics about containers in a central database, and provides a UI for browsing that data.
- `Cluster-level Logging` : A Cluster-level logging mechanism is responsible for saving container logs to a central log store with search/browsing interface.

### Objects

#### Object Spec and Status
Every Kubernetes object includes two nested object fields that govern the object’s configuration: the object spec and the object status. The spec, which you must provide, describes your desired state for the object–the characteristics that you want the object to have. The status describes the actual state of the object, and is supplied and updated by the Kubernetes system. At any given time, the Kubernetes Control Plane actively manages an object’s actual state to match the desired state you supplied.
단에서
#### Describing a Kubernetes Object
When you create an object in Kubernetes, you must provide the object spec that describes its desired state, as well as some basic information about the object (such as a name).

#### Required Fields
In the .yaml file for the Kubernetes object you want to create, you’ll need to set values for the following fields:
- `apiVersion` - Which version of the Kubernetes API you’re using to create this object
- `kind` - What kind of object you want to create
- `metadata` - Data that helps uniquely identify the object, including a  name  string,  UID, and optional  namespace
- `spec` - The precise format of the object  spec is different for every Kubernetes object, and contains nested fields specific to that object.

#### Identifiers and Names
All objects in the Kubernetes REST API are unambiguously identified by a Name and a UID.

#### Namespaces

#### Labels and Selectors

### Workload

#### Pods
A Pod is the smallest deployable object in the Kubernetes object model.

#### Controllers
A Deployment is responsible for creating and updating instances of your application.

### Services, Load Balancing, and Networking

#### Service
An abstract way to expose an application running on a set of Pods as a network service.

#### DNS

#### Ingress

### Storage

#### Volumes
#### Persistent Volumes

## 9. Appendix

#### reference site

* Elasticsearch Reference [7.1] » Modules » Node
https://www.elastic.co/guide/en/elasticsearch/reference/current/modules-node.html

* Kubernetes Concepts
https://kubernetes.io/docs/concepts/

- Overview
	- What is Kubernetes
	- Kubernetes Components
	- The Kubernetes API
	- Working with Kubernetes Objects
		- Understanding Kubernetes Objects
		- Kubernetes Object Management
		- Names
		- Namespaces
		- Labels and Selectors
		- Annotations
		- Field Selectors
		- Recommended Labels
- Kubernetes Architecture
	- Nodes
	- Master-Node communication
	- Concepts Underlying the Cloud Controller Manager
- Containers
	- Images
	- Container Environment Variables
	- Runtime Class
	- Container Lifecycle Hooks
- Workloads
	- Pods
		- Pod Overview
		- Pods
		- Pod Lifecycle
		- Init Containers
		- Pod Preset
		- Disruptions
	- Controllers
		- ReplicaSet
		- ReplicationController
		- Deployments
		- StatefulSets
		- DaemonSet
		- Garbage Collection
		- TTL Controller for Finished Resources
		- Jobs - Run to Completion
		- CronJob
- Services, Load Balancing, and Networking
	- Service
	- DNS for Services and Pods
	- Connecting Applications with Services
	- Ingress
	- Ingress Controllers
	- Network Policies

+ 쿠버네티스 #2 - 개념 이해 (1/2)
https://bcho.tistory.com/1256?category=731548

+ Kubernetes Architecture 101
https://www.aquasec.com/wiki/display/containers/Kubernetes+Architecture+101
