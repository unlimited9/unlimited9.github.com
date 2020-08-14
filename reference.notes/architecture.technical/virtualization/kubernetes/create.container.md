
#### create pod : ggid-dev.build
vi gid-dev.build.yaml
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

`linkerd inject`  
cat ggid-dev.build.yaml | linkerd inject - | kubectl apply -f -

`container`  
kubectl exec -it ggid-dev.build /bin/bash

`delete`  
kubectl delete pod ggid-dev.build

---

`create group/user : app/app`  
groupadd -g 3000 app  
useradd -d /apps -g 3000 -m -u 3000 -s /bin/bash app  
passwd app  
adduser app sudo

`create directory`  
mkdir -p /apps/install /pgms /data /logs
chown -R app.app /apps /pgms /data /logs

su - app
mkdir /pgms/ggcore

cd /apps/install


