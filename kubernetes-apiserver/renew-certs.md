# Renew Certs 

## Zertifikate überprüfen 

```
kubeadm certs check-expiration
```

```
. Wo werden Zertifikate benötigt ?

- zum kube-apiserver hin von den einzelnen Komponenten 
- zum 
usw. 
```

## Sonderrolle 

```
b. Sonderrolle kubelet

Macht ein automatisches Renew the certifikate über die 
Zertifikat api. Schritte:


Es erfolgt ein automatisches Approval des Signing Requests
über den Controller Manager 

Diese muss aktiviert sein:

https://kubernetes.io/docs/tasks/tls/certificate-rotation/
--rotate-certificates 
```

```
root@worker1:/var/lib/kubelet# grep -r "rotate" config.yaml
rotateCertificates: true
```

## Zertifikatserneuerung 

### Schritt 1: 

```
c. Wir erneuern wir Zertifikate ? 

Wichtig: Das muss auf allen Control-Nodes passieren, wenn sie kurz vor dem ablaufen sind.

auf dem controlplane (bspw. api-server) 
kubeadm certs renew apiserver 
```

## Wichtig, kein kubectl delete po verwenden .
command output may be misleading in describing static pods: even if it shows that the static pod restarted recently, the correspondent pod containers were not restarted.

# dann das manifests wegschieben
cd /etc/kubernetes/manifests/
mv kube-apiserver.yaml /tmp 

# will not work anymore, because apiserver is not running
kubectl -n kube-system get pods 
export CONTAINER_RUNTIME_ENDPOINT=unix:///var/run/containerd/containerd.sock
crictl pods

# taucht nicht mehr auf -> ap 
mv /tmp/kube-apiserver.yaml . 
crictl pods | grep api

kubectl get nodes 

kubeadm certs check-expiration | grep "apiserver "
apiserver                  Jan 23, 2025 04:09 UTC   364d            ca                      no

echo | openssl s_client -showcerts -connect  64.226.76.200:6443 -servername api 2>/dev/null | openssl x509 -noout -enddate
notAfter=Jan 23 04:09:04 2025 GMT

d. Zertifikate erneuern ohne Downtime 

Das wird nur funktionieren, wenn mir eine HA-Cluster haben.
Dort gibt es mehrere Controlplanes und wir haben einen LoadBalancer davor.
-> hier vielleicht noch ein Schaubild zeigen.

Ansonsten muss immer der kube-api-server neu gestartet werden und die einzelnen 
Komponenten, hier haben wir immer eine kurze Downtime. 

Dies wird durch ein HA-Cluster vermieden. Dort ist ein LoadBalancer davorgeschaltet.

```
