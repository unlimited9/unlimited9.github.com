create controller : replication controller

```yaml
apiVersion: v1
kind: ReplicationController
metadata:
  name: mobon.product.rc
spec:
  replicas: 3
  selector:
    app: mobon.product
  template:
    metadata:
      name: mobon.product.pod
      labels:
        app: mobon.product
    spec:
      containers:
      - name: mobon.product
        image: mobon/apps.server/1.1
        imagePullPolicy: Always
        ports:
        - containerPort: 8080
```
