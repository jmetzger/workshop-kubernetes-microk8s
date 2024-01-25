# Step-By-Step - Liveness / Readiness Exercise 

## Step 1: Create Deployment + Service (without readiness - check) 

```
cd
mkdir -p manifests/probes-combined
cd manifests/probes-combined 
```

```
# vi 01-deploy.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginxtest
spec:
  replicas: 1
  selector:
    matchLabels:
      app: nginxtest
  template:
    metadata:
      labels:
        app: nginxtest
    spec:
      containers:
      - name: nginx
        image: nginx:latest
   
```

```

# 02-svc.yml
apiVersion: v1
kind: Service
metadata:
  name: nginx-test
spec:
  type: ClusterIP
  ports:
  - port: 80
    protocol: TCP
  selector:
    app: nginxtest 


```


```
# Delete the index.html
kubectl apply -f .
kubectl exec -it deployment/nginxtest -- rm /usr/share/nginx/html/index.html 
```

```
# Endpoint is still available but will produce an 404
kubectl get svc nginx 
kubectl run --rm podtester --image=busybox -- wget -O - http://nginx  

```


## Step 2: Add a readiness - test and re


```
# vi 01-deploy.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginxtest
spec:
  replicas: 1
  selector:
    matchLabels:
      app: nginxtest
  template:
    metadata:
      labels:
        app: nginxtest
    spec:
      containers:
      - name: nginx
        image: nginx:latest
        readinessProbe:
          httpGet:
            path: /
            port: 80
          initialDelaySeconds: 3
          periodSeconds: 3

```


### Step 3: Now also introduce a liveness probe (to have a recreation of container) 

```
# vi 01-deploy.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginxtest
spec:
  replicas: 1
  selector:
    matchLabels:
      app: nginxtest
  template:
    metadata:
      labels:
        app: nginxtest
    spec:
      containers:
      - name: nginx
        image: nginx:latest
        livenessProbe:
          httpGet:
            path: /
            port: 80
            httpHeaders:
          initialDelaySeconds: 3
          periodSeconds: 10
          failureThreshold: 6 # How many loops till it is considere failed // defaults to 3 

        readinessProbe:
          httpGet:
            path: /
            port: 80
          initialDelaySeconds: 3
          periodSeconds: 3


```
