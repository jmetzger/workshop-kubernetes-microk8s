# Prometheus with Grafana (Install with helm)

  * using the kube-

## Step 1: Prepare values-file  

```
cd
mkdir -p manifests 
cd manifests 
mkdir monitoring 
cd monitoring 
```

```
vi values.yml 
```

```
fullnameOverride: prometheus

alertmanager:
  fullnameOverride: alertmanager

grafana:
  fullnameOverride: grafana

kube-state-metrics:
  fullnameOverride: kube-state-metrics

prometheus-node-exporter:
  fullnameOverride: node-exporter
```

## Step 2: Install with helm 

```
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm install prometheus prometheus-community/prometheus -f values.yml --namespace monitoring --create-namespace --version 25.21.0
```

## Step 3: Connect to prometheus from the outside world 

### Step 3.1: Start proxy to connect (to on Linux Client)

```
# this is shown in the helm information 
helm -n monitoring get notes prometheus

# Get pod that runs prometheus 
export POD_NAME=$(kubectl get pods --namespace monitoring -l "app.kubernetes.io/name=prometheus,app.kubernetes.io/instance=prometheus" -o jsonpath="{.items[0].metadata.name}")

# Do the port forwarding 
kubectl -n monitoring port-forward $POD_NAME 9090 &
```

### Step 2.2: Start a tunnel in (from) your local-system to the server 

```
ssh -L 9090:localhost:9090 tln1@164.92.129.7
```

### Step 2.3: Open prometheus in your local browser 

```
# in browser
http://localhost:9090 
```

## Step 3: Connect to the alert manager from the outside world 

### Step 2.1: Start proxy to connect (to on Linux Client)

```
# this is shown in the helm information 
helm -n monitoring get notes prometheus

```


## References:

  * https://github.com/prometheus-community/helm-charts/blob/main/charts/kube-prometheus-stack/README.md
  * https://artifacthub.io/packages/helm/prometheus-community/prometheus

  
