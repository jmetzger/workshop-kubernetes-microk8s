# Which routing mode is used 

```
kubectl -n calico-system describe ds calico-node | grep -A 35  calico-node
# or specific
kubectl -n calico-system describe ds calico-node | egrep -i -e vxlan  -e cluster_type

```

```
Environment:
