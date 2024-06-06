# Pods ausbalancieren (bzw. neu deployen wenn node uncordon 

## Policy, die nach Installation des helm - charts eingerichtet werden muss 

  * Als Möglichkeit: Low Node Utilization 
  * https://github.com/kubernetes-sigs/descheduler?tab=readme-ov-file#lownodeutilization

```
apiVersion: "descheduler/v1alpha2"
kind: "DeschedulerPolicy"
profiles:
  - name: ProfileName
    pluginConfig:
    - name: "LowNodeUtilization"
      args:
        thresholds:
          "cpu" : 20
          "memory": 20
          "pods": 20
        targetThresholds:
          "cpu" : 50
          "memory": 50
          "pods": 50
    plugins:
      balance:
        enabled:
          - "LowNodeUtilization"
```

## Ref:

  * https://github.com/kubernetes-sigs/descheduler
  * https://artifacthub.io/packages/helm/descheduler/descheduler
