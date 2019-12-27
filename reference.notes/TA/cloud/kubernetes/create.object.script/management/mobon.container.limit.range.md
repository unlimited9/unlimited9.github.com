## management

#### mobon.container.limit.range
$ vi management/mobon.container.limit.range.yaml
```
apiVersion: v1
kind: LimitRange
metadata:
  name: mobon
spec:
  limits:
    - default:
        cpu: 1000m
        memory: 2000Mib
      defaultRequest:
        cpu: 500m
        memory: 500Mib
      max:
        cpu: 2000m
        memory: 3000Mib
      min:
        cpu: 10m
        memory: 10Mib
      type: Container

```
