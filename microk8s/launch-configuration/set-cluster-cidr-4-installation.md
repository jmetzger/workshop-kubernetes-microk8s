# Using launch configuration for installation - cluster/service - cidr (from 1.28+) 

## Version without adding node 

### Step 1: Setup /etc/microk8s.yaml on node 

#### Version 1: Without joining (from microk8s 1.28+)

  * Nice, because it sets all the right places 

```
---
version: 0.2.0
extraCNIEnv:
  IPv4_CLUSTER_CIDR: "172.18.0.0/16"
  IPv4_SERVICE_CIDR: "172.17.0.0/24"
extraSANs:
  - 172.17.0.1
addons:
  - name: dns
  - name: rbac 
```

#### Version 2: With persistent cluster token 

  * Always use the same token

```
---
version: 0.2.0
persistentClusterToken: "a74cddf30d2408d49fcd748a26021c6a"
join:
# adjust x to your ip
  url: "10.135.0.x:25000/a74cddf30d2408d49fcd748a26021c6a"   
extraCNIEnv:
  IPv4_CLUSTER_CIDR: "172.18.0.0/16"
  IPv4_SERVICE_CIDR: "172.17.0.0/24"
extraSANs:
  - 172.17.0.1
addons:
  - name: dns
  - name: rbac
```


### Step 2: Installieren von microk8s 

  * Wenn einer fehler in der config ist, installiert er nicht ! (Das ist gut)

```
snap install microk8s --classic 
```

### Step 3: Wiederholen für weitere Nodes 

  * Schritt 1+2 wiederholen für alle weiteren Nodes 


### Troubleshooten (Variante 1) 

  * In einem Fall hatte ich die falsche IP in /etc/microk8s.yaml eingetragen
  * Ich habe diese geändert und dann ein:
  * snap set microk8s config="$(cat /etc/microk8s.yaml)"
  * Er hat das dann nochmal neu eingelesen und durchgeführt 

### Troubleshooten (Variante 2) 

  * Sollte es wider Erwarten nicht funktionieren:

```
1. Nochmal die /etc/microk8s.yaml überprüfen
2. microk8s nochmal komplett deinstallieren:
snap remove microk8s --purge
3. Nochmal neu installieren 
snap install microk8s --classic 
```

### Überprüfen von service-cidr und pod-cidr auf dem system 

  * Ist alles richtig eingetragen
  * Konfigurationen werden in microk8s alle unter /var/snap/microk8s/current/args gespeichert

```
# Das einfachste ist anhand der IP - Range danach zu greppen
cd /var/snap/microk8s/current/args
grep -r "172.17." .
grep -r "172.18." . 
```

## Reference: 

 *  https://github.com/canonical/microk8s-cluster-agent/blob/main/pkg/k8sinit/testdata/schema/full.yaml
