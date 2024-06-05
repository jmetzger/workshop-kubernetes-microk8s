# Redirect from http -> https 

## You have to set that in httproute 

```
apiVersion: gateway.networking.k8s.io/v1
kind: HTTPRoute
metadata:
 name: apple-https-route
 annotations:
   konghq.com/protocols: "https"
   konghq.com/https-redirect-status-code: "302"
spec:
  parentRefs:
  - name: prod-https-kong
  hostnames:
  - gimmeyourapple.app6.do.t3isp.de
  rules:
  - backendRefs:
    - name: apple-service-kong
      port: 80


```
