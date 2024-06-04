# Traefik - Gateway - API 

## Important 

```
# As of 01.06.2024, the support for gateway api is currently experimental
# Feature of Gateway API v1.0.0 are only supported partly
```

## Prerequisites 

  * You need to have LoadBalancers in your clusters
  * In our case, we used Metallb with DigitalOcean
    * [Metallb installieren](metallb/metallb-helm-l2-aufsetzen.md)
   
## Step 1: Install (we are doing this all in our default namespace)

```
# gateway and httproute need to be in the same namespace by default
```

```
# Important you need to activated the experimenal gateway api provider
# Information in some places in unfortunately not complete / correct 
```

```
helm repo add traefik https://traefik.github.io/charts
helm install traefik --set experimental.kubernetesGateway.enabled=true traefik/traefik --version 28.2.0
# This installs the gatewayclass and the gateway
```

## Step 2: Setup a test project (deploy and service) 

```
mkdir -p manifests
cd manifests
mkdir traefik-sample
cd traefik-sample
```

```
vi 01-whoami-deploy-and-service.yml
```

```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: whoami

spec:
  replicas: 2
  selector:
    matchLabels:
      app: whoami
  template:
    metadata:
      labels:
        app: whoami
    spec:
      containers:
        - name: whoami
          image: traefik/whoami:v1.6.0
          ports:
            - containerPort: 80
              name: http

---
apiVersion: v1
kind: Service
metadata:
  name: whoami
spec:
  selector:
    app: whoami

  ports:
    - port: 80
      targetPort: http

```

```
kubectl apply -f .
```

## Step 3: Setup httproute 

```
vi 02-httproute.yaml 
```

```
apiVersion: gateway.networking.k8s.io/v1
kind: HTTPRoute
metadata:
  name: http-app-1
  namespace: default

spec:
  parentRefs:
  - name: traefik-gateway

  rules:
  - backendRefs:
    - name: whoami
      port: 80
```

```
kubectl apply -f .
```

## Step 4: Testit  

```
# Find the External ip in the browser
kubectl get svc traefik 
```

```
# output
NAME      TYPE           CLUSTER-IP    EXTERNAL-IP       PORT(S)                      AGE
traefik   LoadBalancer   172.17.0.36   157.230.113.124   80:31491/TCP,443:32010/TCP   55m
```

```
# open in your favourite browser
# you should see the whoami page now 
http://157.230.113.124/
```

## Entrypoints 

```
# Gateway uses entrypoints, which need to be configured in
# So the easiest way, would be to use exactly these entrypoints
# Default entrypoint for web: 8000
# (Although opened to the outside world with 80)
kubectl get gateway traefik-gateway -o yaml 
```

```
# output
kind: Gateway
  name: traefik-gateway
  namespace: default
spec:
  gatewayClassName: traefik
  listeners:
  - allowedRoutes:
      namespaces:
        from: Same
    name: web
    port: 8000
    protocol: HTTP
```

```
# How can find out which entrypoints are opened in traeffik
kubectl describe deploy/traefik | grep -A 20 "Args:"
```

```
# Output
Args:
      --global.checknewversion
      --global.sendanonymoususage
      --entryPoints.metrics.address=:9100/tcp
      --entryPoints.traefik.address=:9000/tcp
      --entryPoints.web.address=:8000/tcp
      --entryPoints.websecure.address=:8443/tcp
      --api.dashboard=true
      --ping=true
      --metrics.prometheus=true
      --metrics.prometheus.entrypoint=metrics
      --providers.kubernetescrd
      --providers.kubernetesingress
      --providers.kubernetesgateway
      --experimental.kubernetesgateway
      --entryPoints.websecure.http.tls=true
      --log.level=INFO
```

## Investigate 

  * If you want to investigage how everyhing is tied together look here:

```
helm pull traefik/traefik
tar xzvf traefik*.tar.gz
cd traefik/templates
cat gateway.yaml
cat gatewayclass.yaml
```
