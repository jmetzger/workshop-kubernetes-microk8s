# Example tcp routing 

## Prerequisites: 
 
  * Remove helm chart for ingress if present : helm -n kong uninstall kong (if ingress was installed as kong) 
  * kong - chart is installed
    * https://artifacthub.io/packages/helm/kong/kong
    * and a service runs for kong-gateway runs in kong - namespace

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
kubectl -n kong get svc kong-kong-proxy
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
apiVersion: gateway.networking.k8s.io/v1
kind: Gateway
metadata:
  name: prod-stream-9000
spec:
  gatewayClassName: kong
  listeners:
  - protocol: TCP
    port: 9000
    name: stream9000
```

```
kubectl apply -f .
kubectl get gateway prod-stream-9000
```

## Step 7: Setup tcprouting 

```
vi 04-tcprouting.yaml
```

```
apiVersion: gateway.networking.k8s.io/v1alpha2
kind: TCPRoute
metadata:
  name: echo-plaintext
spec:
  parentRefs:
  - name: prod-stream-9000
    sectionName: stream9000
  rules:
  - backendRefs:
    - name: echo
      port: 1025
```

```
kubectl apply -f .
```

## Step 8: Verify 

### Verification 1:

```
# Is there a status ?
kubectl describe tcproute echo-plaintext

# If there is not status.....
# -> Objekt is in etcd -> BUT: it is not processed by Kong
```

## Step 9: Fix 

```
# We are using an alpha - feature, and need to enable these alpha - features in Kong
https://docs.konghq.com/kubernetes-ingress-controller/latest/reference/feature-gates/
```

```
# Version 1: Just for testing
kubectl set env -n kong deployment/kong-kong CONTROLLER_FEATURE_GATES="GatewayAlpha=true" -c ingress-controller
```

```
# Version 2: Set it correctly in your kong-values.yaml file and upgrade
## add 
ingressController:
  env:
    feature_gates: FillIDs=true,RewriteURIs=true
```

```
## i have not tested this yet
helm -n kong upgrade kong kong/kong -f kong-values.yaml 
```

## Step 10: Retest 

```
kubectl -n kong describe tcproute echo-plaintext
kubectl get tcproute echo-plaintext -ojsonpath='{.status.parents[0].conditions[?(@.reason=="Accepted")]}'
```

## Step 11: Test connection 

```
# You should get the public ip from here 
kubectl -n kong get svc 
# on your local machine our linux-client
telnet <public-ip-of-kong-proxy> 9000 

# now you can write something and
# and it will get returned back 
```

## Reference 

  * https://docs.konghq.com/kubernetes-ingress-controller/3.1.x/guides/services/tcp/
  * https://gateway-api.sigs.k8s.io/guides/tcp/
