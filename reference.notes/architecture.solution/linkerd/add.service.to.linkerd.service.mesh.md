## Add service to linkerd service mesh

#### linkerd inject
cat service.yaml | linkerd inject - | kubectl apply -f -  

>linkerd injected
```
...
spec:
  ...
  template:
    metadata:
      annotations:
        linkerd.io/inject: enabled
...
```

## appendix

#### reference.site

* Adding Your Service  
https://linkerd.io/2/tasks/adding-your-service/  
