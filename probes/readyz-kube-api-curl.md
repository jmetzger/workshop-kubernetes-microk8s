# Prüfen des readyz endpoints vom Kube Api Server 

## Walkthrough 

```
kubectl run -it --rm podtester --image=busybox 
# im pod 
# um zu sehen mit welchem Port wir uns verbinden können 
env | grep -i kubernetes 
# kubernetes liegt als service vor 
wget -O - https://kubernetes:443/readyz?verbose
```

## Reference:

  * https://kubernetes.io/docs/reference/using-api/health-checks/
