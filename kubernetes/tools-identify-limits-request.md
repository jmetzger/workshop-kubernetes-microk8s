# Limits and Requests (Tools to identify) 

## VPA (Vertical Pod Autoscaler) / goldilocks 

```
# Please only repo updateMode: "off" will do this 
# Do not use automatic adjustment 
Example VPA configuration
apiVersion: autoscaling.k8s.io/v1
kind: VerticalPodAutoscaler
metadata:
  name: my-app-vpa
spec:
  targetRef:
    apiVersion: "apps/v1"
    kind:       Deployment
    name:       my-app
  updatePolicy:
    updateMode: "off"
```

  * goldilocks will now make visible instead of kubectl describe vpa 
  * https://github.com/FairwindsOps/goldilocks
  * als Basis: https://github.com/kubernetes/autoscaler/
  * https://www.fairwinds.com/goldilocks

