# Hängenden Namespace löschen 

```
kubectl get namespace "metallb-system" -o json   | tr -d "\n" | sed "s/\"finalizers\": \[[^]]\+\]/\"finalizers\": []/"  | kubectl replace --raw /api/v1/namespaces/metallb-system/finalize -f -
```
