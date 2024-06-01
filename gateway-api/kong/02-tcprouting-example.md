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

## Step 3: Setup gatewayclass 

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

## Step 4: Beispiel echo-service 

```
wget -O 02-echo-service.yaml https://docs.konghq.com/assets/kubernetes-ingress-controller/examples/echo-service.yaml
cat 02-echo-service.yaml
kubectl apply -f 02-echo-service.yaml 
```

## Step 5: Test it locally 
se
```
kubectl get svc echo
# show the ports e.g. 1025
kubectl run -it --rm podtester --image=busybox
```

```
telnet echo 1025
# you can write something and will get it back
hello you, Jochen
hello you, Jochen
```

```
CRTL + C
e
```

## Step 6: Setup gateway 

```
vi 03-gateway.yml
```

```





## Step 5: Setup tcprouting 

```
vi 03-tcprouting.yaml
```

```




## Reference 

  * Example from envoy as reference: 
