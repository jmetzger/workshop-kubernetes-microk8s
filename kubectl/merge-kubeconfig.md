# Mehrere Kubeconfigs in eine kubeconfig mergen 

## So funktioniert es auch bereits:

```
# hier werden mehrere kubeconfigs durchsucht 
export KUBECONFIG=~/.kube/config:/path/cluster1:/path/cluster2

```

## Jetzt alles in eine Datei 

```
cd ~/.kube
kubectl config view --flatten > all-in-one-kubeconfig.yaml
mv config config.old 
mv all-in-one-kubeconfig.yaml config
```

## Contexts jeweils anzeigen 

```
kubectl config 
kubectl config use-context mycontext
```
