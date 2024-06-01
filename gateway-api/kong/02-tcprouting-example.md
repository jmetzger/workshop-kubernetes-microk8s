# Example tcp routing 

## Prerequisites: 

  * kong is installed and a service runs for kong-gateway runs in kong - namespace

```
kubectl -n kong get svc kong-gateway-proxy
```

## Step 1: Setup experimental crds 
 

```
cd
mkdir -p manifests
cd manifests
mkdir -p kong-gateway-tcp
cd kong-gateway-tcp
mkdir api
cd api
```

```
wget -O gateway-api-1.1.0-experimental.yml https://github.com/kubernetes-sigs/gateway-api/releases/download/v1.1.0/experimental-install.yaml
kubectl apply -f .
```

### Step 2: setup kong 


```
vi kong-values.yml
```

```
proxy:
  stream:
    - containerPort: 9000
      servicePort: 9000
      protocol: TCP
```

```
helm repo add kong https://charts.konghq.com
helm install kong kong/kong -f kong-values.yml --namespace kong --create-namespace
```

```
# Verify that listen is setup
kubectl -n kong get svc kong-kong-gateway
kubectl -n kong describe deploy kong-kong | grep KONG_STREAM_LISTEN
```


## Reference 

  * Example from envoy as reference: 
