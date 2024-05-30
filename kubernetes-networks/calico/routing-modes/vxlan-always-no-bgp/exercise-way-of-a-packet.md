# Exercise way of a packet 

## What we want to check ? 

  * We want to see the way of a packet 

## Voraussetzung 

  * Setup ist: microk8s (1.29) & calico 
  * calico läuft als vxlan: always
  * calico hat kein BGP und kein eBPF aktiviert

## Schritt 1: Vorbereitung ( 2 Pods auf unterschiedlichen nodes starten )

```
kubectl get nodes
```

```
# Achtung: Nodes anpassen
NODE1=worker1
NODE2=worker2
```

```
# Node 1 
kubectl apply -f - <<EOF
apiVersion: v1
kind: Pod
metadata:
  labels:
    app: curl
  name: curl
spec:
  containers:
  - image: curlimages/curl
    name: curl
    command: ["sleep", "365d"]
  nodeName: $NODE1
---
# Node 2 
# httpbin antwortet einfach auf meine http anfragen 
apiVersion: v1
kind: Pod
metadata:
  labels:
    app: httpbin
  name: httpbin
spec:
  containers:
  - image: kennethreitz/httpbin
    name: httpbin
  nodeName: $NODE2
EOF

```

## Schritt 2: Pod-IP des Ziels ausfindig machen und aufrufen. 

```
kubectl get po httpbin -o wide
```

```
# Beispielausgabe
172.18.104.14
```

## Schritt 3: Ziel-Pod (httpbin auf NODE2) von curl auf NODE1 aus aufrufen

```
# Verbindung funktioniert 
kubectl exec -it curl -- curl http://172.18.104.14:80/
```

## Schritt 3.1: CURL Debuggen 1: Lass uns die Route zu 172.18.104.14 herausfinden (von curl aus) 

```
kubectl exec curl -- ip route get 172.18.104.14
```

```
# Beispielausgabe
172.18.104.14 via 169.254.1.1 dev eth0  src 172.18.166.139
```

```
## ERKLÄRUNG 
# <- 169.254.1.1 ist hier eine spezielle Adresse die proxy_arp verwenden
# Alle IP-Adressanfragen, die ausserhalb des Netzes sind, werden
# an den proxy_arp proxy geschickt
# Das sind im System alle Elemente der calico-veth-Paare, die auf dem Hostsystem sind:
```

```
# So finden wir diese raus:
kubectl debug node/node1 -it --image=busybox
```

```
for i in  /host/proc/sys/net/ipv4/conf/cali*; do echo $i; cat $i/proxy_arp; done
```

```
/host/proc/sys/net/ipv4/conf/cali03d459aac7e
1
/host/proc/sys/net/ipv4/conf/cali35e3a5cb80b
1
/host/proc/sys/net/ipv4/conf/cali62facc760af
1
```

## Schritt 3.2 Aber welches Paar ist es nun ? 

```
# Im Container:
kubectl exec -it curl -- ip a eth0
```

```
# Die Ausgabe:
3: eth0@if19: <BROADCAST,MULTICAST,UP,LOWER_UP,M-DOWN> mtu 1450 qdisc noqueue state UP
    inet 172.18.166.139/32 scope global eth0
```

```
# Das Equivalent findet sich auf dem Server
kubectl debug -it node/node1 --image=busybox
```

```
ip a | grep ^19
```

```
# Ausgabe 
19: cali35e3a5cb80b@eth1: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1450 qdisc noqueue
```

```
# Yeah, das ist unser veth - paar
```





## Schritt 3.2. CURL: Debuggen 2: Wie ist die Mac-Adresse von 10.244.0.237 

  * **Wichtig:** Wir wollen über eth0 des containers rausgehen.

```
# Lasst uns die Mac der IP herausfinden
kubectl exec curl  -- arp -n
```

```
# Beispielausgabe
? (10.244.0.237) at f6:25:f2:9e:91:49 [ether]  on eth0
```

## Schritt 3.3 CURL Debuggen 3: Ist das die Mac-Adresse des eth0 interfaces im Container ? 

```
# Herausgefunden haben wir als Beispiel
f6:25:f2:9e:91:49 [ether]  on eth0
```

```
# Das ist aber die MAC des eth0 - Adapter im pod (curl) 
kubectl exec curl -- ip link show eth0
```

```
# Ausgabe: / OH eine ganz andere MAC ! 
256: eth0@if257: <BROADCAST,MULTICAST,UP,LOWER_UP,M-DOWN> mtu 1500 qdisc noqueue state UP qlen 1000
    link/ether 9a:b4:aa:83:18:62 brd ff:ff:ff:ff:ff:ff
```

  * **ACHTUNG** - andere MAC und Teil eines veth - Paares (eth0@if257)- der andere Teil ist das Interface 257 auf dem Host (Worker Node) 

## Schritt 3.4 Das Interface auf dem Host-System finden 

```
kubectl debug -it node/$NODE1 --image=busybox -- sh
```

```
# in der shell nr bitte anpassen
ip link | grep -A1 ^257 
```

```
# Beispielausgabe # OK !!! gleiche MAC wie aus Schritt 3.3. 
257: lxced3096cb2cb1@if256: <BROADCAST,MULTICAST,UP,LOWER_UP,M-DOWN> mtu 1500 qdisc noqueue qlen 1000
    link/ether f6:25:f2:9e:91:49 brd ff:ff:ff:ff:ff:ff
```

  *  **Calico hijacks ARP (proxy_arp) table of POD1, forces the next hop to be the peer end (host side) of the veth pair.**


```
kubectl get nodes -o wide 
```

## Schritt 4: On Node 2: 



## Reference:

  * https://addozhang.medium.com/kubernetes-network-learning-with-cilium-and-ebpf-aafbf3163840
