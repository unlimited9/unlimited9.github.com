
#### mermaid
https://mermaid-js.github.io/mermaid

#### mermaid : live editor
https://mermaid-js.github.io/mermaid-live-editor


`graph directions`  
TB - top bottom  
BT - bottom top  
RL - right left  
LR - left right  
TD - same as TB  

`sample flowchart`  
```mermaid
graph LR
  %% define object
  User(Audience)
  subgraph DMZ
    L4((tk.mediacategory.com<br/>api.mediacategory.com<br/><br/>VIP : 119.205.238.104))
    Proxy01[MPK-WEB-01<br/><br/>RIP : 119.205.238.105 : 10.251.0.180]
    Proxy02[MPK-WEB-02<br/><br/>RIP : 119.205.238.106 : 10.251.0.181]
  end
  style DMZ fill:none,stroke:#333,stroke-width:2px
  subgraph Kubernetes
    subgraph MPK-Cluster
      Gateway((Service Gateway<br/><br/>- mobon-gateway-api-common<br/>: 10.251.0.183<br/>- mobon-gateway-service-aggregation<br/>: 10.251.0.184))
    end
  end
  style Kubernetes fill:none, stroke:#333,stroke-width:2px
  style MPK-Cluster fill:#FDF5E6,stroke:#333
  
  %% define object style class
  classDef proxyClass fill:#dcdcdc,stroke:#333

  class L4 proxyClass
  Gateway:::proxyClass

  %% define link
  User --WAN--> L4
  L4 --> Proxy01
  L4 --> Proxy02
  Proxy01 --> Gateway
  Proxy02 --> Gateway
```  
