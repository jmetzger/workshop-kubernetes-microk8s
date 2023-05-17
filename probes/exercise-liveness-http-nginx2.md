# Liveness http-exercise to nginx 

## Walkthrough 

```
cd 
mkdir -p manifests 
cd manifests
mkdir live-http
cd live-http
vi 01-pod.yaml 
```


```
apiVersion: v1
kind: Pod
metadata:
  name: liveness-http-demo
spec:
  containers:
  - name: nginx
    image: nginx
    ports:
    - containerPort: 80
    volumeMounts:
    - name: workdir
      mountPath: /usr/share/nginx/html
    livenessProbe:
      initialDelaySeconds: 2
      periodSeconds: 5
      httpGet:
        path: /testpage.html
        port: 80
  # These containers are run during pod initialization
  initContainers:
  - name: install
    image: busybox:1.28
    command:
    - wget
    - "-O"
    - "/work-dir/testpage.html"
    - http://info.cern.ch
    volumeMounts:
    - name: workdir
      mountPath: "/work-dir"
  dnsPolicy: Default
  volumes:
  - name: workdir
    emptyDir: {}
```

```
kubectl apply -f . 
kubectl exec -it liveness-http-demo -- bash 
# mv testpage.html in pod 
mv /usr/share/nginx/html/testpage.html /usr/share/nginx/html/gone.html 
```

```
# now look into pod 
# you will see restarts and a crash loop after a while 
kubectl get pods -w liveness-http-demo 
```
    
