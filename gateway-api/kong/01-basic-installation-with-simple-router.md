# Install / Prepare Gateway API with kong 

## Step 1: Install CRD's for Gateway API 

```
cd
mkdir -p manifests
cd manifests
mkdir -p kong-gateway
cd kong-gateway
```

```
wget -O 00-gateway-api-standard-1.1.0.yml https://github.com/kubernetes-sigs/gateway-api/releases/download/v1.1.0/standard-install.yaml
kubectl apply -f 00-gateway-api-standard-1.1.0.yml
```

## Step 2: Install kong with helm 

```
helm install kong --namespace kong --create-namespace --repo https://charts.konghq.com ingress
```


### Step 3: Setup Gateway Class for Kong 

```
vi 01-gatewayclass.yml
```

```
apiVersion: gateway.networking.k8s.io/v1
kind: GatewayClass
metadata:
  name: kong
  annotations:
    konghq.com/gatewayclass-unmanaged: 'true'

spec:
  controllerName: konghq.com/kic-gateway-controller
```

```
kubectl apply -f .
```

### Step 4: Setup Gateway for your project 

```
vi 02-gateway.yml
```

```
apiVersion: gateway.networking.k8s.io/v1
kind: Gateway
metadata:
  name: prod-web-kong
spec:
  gatewayClassName: kong
  listeners:
  - protocol: HTTP
    port: 80
    name: prod-web-gw
    allowedRoutes:
      namespaces:  
        from: Same
```

### Step 5: Setup httproute for gateway

```
vi 03-http-route.yml
```

```
apiVersion: gateway.networking.k8s.io/v1
kind: HTTPRoute
metadata:
  name: apple-http-route
spec:
  parentRefs:
  - name: prod-web-kong
  rules:
  - backendRefs:
    - name: apple-service-kong
      port: 80
```

```
kubectl apply -f .
```

### Step 6: Setup apple-pod and apple-service 

```
vi apple-pod.yml 
```

```
# apple.yml
# vi apple.yml
kind: Pod
apiVersion: v1
metadata:
  name: apple-app-kong
  labels:
    app: apple-kong
spec:
  containers:
    - name: apple-app
      image: hashicorp/http-echo
      args:
        - "-text=apple"
```

```
kubectl apply -f .
```

```
vi 05-apple-service.yml
```

```
# apple.yml
kind: Service
apiVersion: v1
metadata:
  name: apple-service-kong
spec:
  selector:
    app: apple-kong
  ports:
    - protocol: TCP
      port: 80
      targetPort: 5678 # Default port for image
```


```
kubectl apply -f .
```

### Step 7: Test it 

```
# Find out the public ip
kubectl -n kong get svc kong-gateway
```

```
# In your browser open
# this ip from above
```

### Reference / Get Started 

  * https://github.com/kong/kubernetes-ingress-controller
  * https://gateway-api.sigs.k8s.io/guides/simple-gateway/
