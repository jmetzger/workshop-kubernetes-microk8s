# Debug Container / Debug Node 

##  Walkthrough  Debug Container 

```
# Show that is does not work 
kubectl run ephemeral-demo --image=registry.k8s.io/pause:3.1 --restart=Never
kubectl exec -it ephemeral-demo -- sh

# Start a debug container 
kubectl debug -it ephemeral-demo --image=busybox 
```

## Walkthrough Debug Node 

```
kubectl get nodes 
kubectl debug node/mynode -it --image=ubuntu
```



## Reference 

  * https://kubernetes.io/docs/tasks/debug/debug-application/debug-running-pod/#ephemeral-container
