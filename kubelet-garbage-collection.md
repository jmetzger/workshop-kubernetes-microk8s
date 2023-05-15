# kubelet garbage collection 

## What is do  ? 

 * Deletes unused containers after 1 minutes 
 * and unused images after 5 minutes 

## Reference:

  * https://kubernetes.io/docs/concepts/architecture/garbage-collection/#containers-images

## list images with ctr 

  * ctr is the cli tool for containerd 

```
# from client  
kubectl run nginx --image nginx 

# on worker - node 
ctr images list | grep nginx 
```
