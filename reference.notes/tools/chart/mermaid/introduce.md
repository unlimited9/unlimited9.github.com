
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
https://mermaid-js.github.io/mermaid-live-editor/#/edit/eyJjb2RlIjoiZ3JhcGggTFJcbiAgJSUgZGVmaW5lIG9iamVjdFxuICBVc2VyKEF1ZGllbmNlKVxuICBzdWJncmFwaCBETVpcbiAgICBMNCgodGsubWVkaWFjYXRlZ29yeS5jb208YnIvPmFwaS5tZWRpYWNhdGVnb3J5LmNvbTxici8-PGJyLz5WSVAgOiAxMTkuMjA1LjIzOC4xMDQpKVxuICAgIFByb3h5MDFbTVBLLVdFQi0wMTxici8-PGJyLz5SSVAgOiAxMTkuMjA1LjIzOC4xMDUgOiAxMC4yNTEuMC4xODBdXG4gICAgUHJveHkwMltNUEstV0VCLTAyPGJyLz48YnIvPlJJUCA6IDExOS4yMDUuMjM4LjEwNiA6IDEwLjI1MS4wLjE4MV1cbiAgZW5kXG4gIHN0eWxlIERNWiBmaWxsOm5vbmUsc3Ryb2tlOiMzMzMsc3Ryb2tlLXdpZHRoOjJweFxuICBzdWJncmFwaCBLdWJlcm5ldGVzXG4gICAgc3ViZ3JhcGggTVBLLUNsdXN0ZXJcbiAgICAgIEdhdGV3YXkoKFNlcnZpY2UgR2F0ZXdheTxici8-PGJyLz4tIG1vYm9uLWdhdGV3YXktYXBpLWNvbW1vbjxici8-OiAxMC4yNTEuMC4xODM8YnIvPi0gbW9ib24tZ2F0ZXdheS1zZXJ2aWNlLWFnZ3JlZ2F0aW9uPGJyLz46IDEwLjI1MS4wLjE4NCkpXG4gICAgZW5kXG4gIGVuZFxuICBzdHlsZSBLdWJlcm5ldGVzIGZpbGw6bm9uZSwgc3Ryb2tlOiMzMzMsc3Ryb2tlLXdpZHRoOjJweFxuICBzdHlsZSBNUEstQ2x1c3RlciBmaWxsOiNGREY1RTYsc3Ryb2tlOiMzMzNcbiAgXG4gICUlIGRlZmluZSBvYmplY3Qgc3R5bGUgY2xhc3NcbiAgY2xhc3NEZWYgcHJveHlDbGFzcyBmaWxsOiNkY2RjZGMsc3Ryb2tlOiMzMzNcblxuICBjbGFzcyBMNCBwcm94eUNsYXNzXG4gIEdhdGV3YXk6Ojpwcm94eUNsYXNzXG5cbiAgJSUgZGVmaW5lIGxpbmtcbiAgVXNlciAtLVdBTi0tPiBMNFxuICBMNCAtLT4gUHJveHkwMVxuICBMNCAtLT4gUHJveHkwMlxuICBQcm94eTAxIC0tPiBHYXRld2F5XG4gIFByb3h5MDIgLS0-IEdhdGV3YXlcbiAgXG4iLCJtZXJtYWlkIjp7InRoZW1lIjoiZGVmYXVsdCJ9LCJ1cGRhdGVFZGl0b3IiOmZhbHNlfQ
