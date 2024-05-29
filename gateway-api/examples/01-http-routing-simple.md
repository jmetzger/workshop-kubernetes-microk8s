# Simple Routing 

## Voraussetzung
  * Gateway API CRD's & Nginx Fabric Gateway müssen installiert sein
  * siehe [Installation nginx](/gateway-api/installation/nginx.md)

## Schritt 1: Gateway aufsetzen (Beispiel mit nginx) 

  * An dieses Gateway kann ich nur Routen im gleichen Namespace anhängen

```
cd
mkdir -p manifests
cd manifests
mkdir -p gateway-nginx-simple 
cd gateway-nginx-simple
```

```
vi 01-gateway.yml
```

```
apiVersion: gateway.networking.k8s.io/v1
kind: Gateway
metadata:
  name: prod-web
spec:
  gatewayClassName: example
  listeners:
  - protocol: HTTP
    port: 80
    name: prod-web-gw
    allowedRoutes:
      namespaces:
        from: Same
```

```
kubectl apply -f .
```

## Schritt 2: HTTP-Route definieren 

```
vi 02-httproute.yml
```

```
apiVersion: gateway.networking.k8s.io/v1
kind: HTTPRoute
metadata:
  name: foo
spec:
  parentRefs:
  - name: prod-web
  rules:
  - backendRefs:
    - name: apple-service
      port: 80

```

```
kubectl apply -f .
```





## Reference:

  * https://gateway-api.sigs.k8s.io/guides/simple-gateway/

