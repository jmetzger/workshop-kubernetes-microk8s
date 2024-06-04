# Simple Routing 

## Voraussetzung
  * Gateway API CRD's & Nginx Fabric Gateway müssen installiert sein
  * siehe [Installation nginx](/gateway-api/installation/nginx.md)

## Schritt 1: Gateway Api Ressources installieren 

```
# Standard - Stabile Features 
kubectl apply -f https://github.com/kubernetes-sigs/gateway-api/releases/download/v1.0.0/standard-install.yaml

# Es gibt auch einen experimentellen Channel, der aber anders installiert werden muss
# MACHEN WIR NICHT 
# kubectl apply -f https://github.com/kubernetes-sigs/gateway-api/releases/download/v1.0.0/experimental-install.yaml
```

## Schritt 2: nginx fabric gateway mit helm installieren 

```
helm install ngf oci://ghcr.io/nginxinc/charts/nginx-gateway-fabric --create-namespace -n nginx-gateway
```

## Schritt 3: Gateway aufsetzen (Beispiel mit nginx) 

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
  gatewayClassName: nginx
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

## Schritt 4: HTTP-Route definieren 

```
vi 02-httproute.yml
```

```
apiVersion: gateway.networking.k8s.io/v1
kind: HTTPRoute
metadata:
  name: apple-http-route
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

## Schritt 4: Service und Pod / Deployment definieren 

```
vi 03-apple-pod.yml
```

```
# apple.yml 
# vi apple.yml 
kind: Pod
apiVersion: v1
metadata:
  name: apple-app
  labels:
    app: apple
spec:
  containers:
    - name: apple-app
      image: hashicorp/http-echo
      args:
        - "-text=apple"
```

```
vi 04-service.yml
```

```
kind: Service
apiVersion: v1
metadata:
  name: apple-service
spec:
  selector:
    app: apple
  ports:
    - protocol: TCP
      port: 80
      targetPort: 5678 # Default port for image
```

```
kubectl apply -f .
```

## Reference: 

  * https://github.com/nginxinc/nginx-gateway-fabric
  * https://docs.nginx.com/nginx-gateway-fabric/installation/expose-nginx-gateway-fabric/

  * https://gateway-api.sigs.k8s.io/guides/simple-gateway/

