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
helm 
helm install traefik --set experimental.kubernetesGateway.enabled=true traefik/traefik
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

```

```
kubectl apply -f .
```

## Step 3: Setup httproute 

```
vi 
```

```




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
