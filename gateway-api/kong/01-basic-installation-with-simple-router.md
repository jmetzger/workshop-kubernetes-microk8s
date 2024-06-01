# Install / Prepare Gateway API with kong 

## Step 1: Install CRD's for Gateway API 

```
cd
mkdir manifests
cd manifests
mkdir kong-gateway
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



### Reference / Get Started 

  * https://github.com/kong/kubernetes-ingress-controller
