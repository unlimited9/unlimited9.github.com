# Exposing the Dashboard

#### Nginx with basic auth

vi linkerd.web.ingress.yaml
```yaml
apiVersion: v1
kind: Secret
type: Opaque
metadata:
  name: web-ingress-auth
  namespace: linkerd
data:
  auth: YWRtaW46JGFwcjEkbjdDdTZnSGwkRTQ3b2dmN0NPOE5SWWpFakJPa1dNLgoK
---
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: web-ingress
  namespace: linkerd
  annotations:
    kubernetes.io/ingress.class: 'nginx'
    nginx.ingress.kubernetes.io/upstream-vhost: $service_name.$namespace.svc.cluster.local:8084
    nginx.ingress.kubernetes.io/configuration-snippet: |
      proxy_set_header Origin "";
      proxy_hide_header l5d-remote-ip;
      proxy_hide_header l5d-server-id;
    nginx.ingress.kubernetes.io/auth-type: basic
    nginx.ingress.kubernetes.io/auth-secret: web-ingress-auth
    nginx.ingress.kubernetes.io/auth-realm: 'Authentication Required'
spec:
  rules:
    - host: dashboard.linkerd.hajimaro.com
      http:
        paths:
          - backend:
              serviceName: linkerd-web
              servicePort: 8084
```
kubectl apply -f linkerd.web.ingress  


## appendix

#### reference.site

* Exposing the Dashboard  
https://linkerd.io/2/tasks/exposing-dashboard/  
