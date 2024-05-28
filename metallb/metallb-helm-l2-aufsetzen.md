# Metallb mit helm für L2 aufsetzen 

## Schritt 1: helm chart installieren 

```
helm repo add metallb https://metallb.github.io/metallb
helm install metallb metallb/metallb --namespace=metallb-system --create-namespace
```

```
# Wait for all the systems to come up
kubectl -n metallb-system get pods -o wide 
```

## Schritt 2: Addressbereich und Propagation-Art konfigurieren 

```
cd
mkdir -p manifests
cd manifests
mkdir lb
cd lb
vi 01-addresspool.yml 
```

```
apiVersion: metallb.io/v1beta1
kind: IPAddressPool
metadata:
  name: first-pool
  namespace: metallb-system
spec:
  addresses:
  # we will use our external ip here 
  - 134.209.231.154-134.209.231.154
  # both notations are possible 
  - 157.230.113.124/24
```

```
kubectl apply -f .
```

```
vi 02-advertisement.yml
```

```
apiVersion: metallb.io/v1beta1
kind: L2Advertisement
metadata:
  name: example
  namespace: metallb-system
```

```
kubectl apply -f .
```

## Schritt 3: Testen.. Bekomme ich eine IP vom LoadBalancer ? 

