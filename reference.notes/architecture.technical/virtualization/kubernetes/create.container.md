
#### Create Pod : ggid-dev.build

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: ggid-dev.build
  labels:
    service-name: ggid-dev.build
spec:
  containers:
  - name: ggid-dev-build-3-1-bionic
    image: mcr.microsoft.com/dotnet/core/sdk:3.1-bionic
    imagePullPolicy: Always
    command: ['sh', '-c', 'tail -f /dev/null']
```
