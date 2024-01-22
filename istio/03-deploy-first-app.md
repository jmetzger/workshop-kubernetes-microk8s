# Deploy first app (Exercise for group) 

## Overview (what we want to do) 

![image](https://github.com/jmetzger/training-kubernetes-advanced/assets/1933318/285fc65a-57ec-425f-bcd7-729777f79a7d)

  * Catalog Service is reachable through api

## Step 1: Vorbereitung - repo mit beispielen klonen 

```
cd
git clone https://github.com/jmetzger/istio-exercises/
cd istio-exercises 
```

## Step 2: Eigenen Namespace erstellen 

```
# Jeder Teilnehmer erstellt seinen eigenen Namespace
# z.B. istioapp-tlnx
# d.h. für Teilnehmer 5 (tln5) -> istioapp-tln5
kubectl create ns istioapp-tln5 
# Context so einstellen, dass dieser namespace verwendet
kubectl config set-context --current --namespace istioapp-tln5 
```

## Step 3: Anwendung untersuchen / istioctl kube-inject 

  * Ihr könnt unten direkt den Pfad nehmen, das ist einfacher ;o) 

```
apiVersion: v1
kind: ServiceAccount
metadata:
  name: catalog
---
apiVersion: v1
kind: Service
metadata:
  labels:
    app: catalog
  name: catalog
spec:
  ports:
  - name: http
    port: 80
    protocol: TCP
    targetPort: 3000
  selector:
    app: catalog
---
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app: catalog
    version: v1
  name: catalog
spec:
  replicas: 1
  selector:
    matchLabels:
      app: catalog
      version: v1
  template:
    metadata:
      labels:
        app: catalog
        version: v1
    spec: 
      serviceAccountName: catalog
      containers:
      - env:
        - name: KUBERNETES_NAMESPACE
          valueFrom:
            fieldRef:
              fieldPath: metadata.namespace
        image: istioinaction/catalog:latest
        imagePullPolicy: IfNotPresent
        name: catalog
        ports:
        - containerPort: 3000
          name: http
          protocol: TCP
        securityContext:
          privileged: false
```

```
# schauen wir uns das mal mit injection an 
istiocl kube-inject -f services/catalog/kubernetes/catalog.yaml | less 
```

## Step 4: Automatische Injection einrichten. 

```
# kubectl label namespace istioapp-tlnx istio-injection=enabled 
# z.B
kubectl label namespace istioapp-tln1 istio-injection=enabled 
```

## Step 5: catalog ausrollen 

```
kubectl apply -f services/catalog/kubernetes/catalog.yaml
```

```
# Prüfen, ob wirklich 2 container in einem pod laufen,
# dann funktioniert die Injection
# WORKS, Yeah !
kubectl get pods 
```

## Step 6: Now you will try to reach that catalog 

```
# do it from your namespace, e.g. tlnx 
# z.B. 
kubectl -n tln1 run -it --rm curly --image=curlimages/curl -- sh
```

````
# within shell of that pod
# catalog.yourappnamespace/items/1
curl http://catalog.istioapp-tln1/items/1
exit
```

