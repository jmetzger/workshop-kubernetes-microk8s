# Overview 

## Features 

  * Responsibility separation

## Komponenten 

  1. GatewayController (Kong, Nginx, Traefik, HAProxy, Istio) - Software
  1. GatewayClass
  1. Gateway 
  1. HttpRoute (GA) / TCPRoute (experimentell) / gRPCRoute (experimentell) 


4. Different types of routing:
httproute
tcproute
grpcroute

5. TrafficRouting based on Request Header

6. Load Balancing




## Shared responsibility 

### Gateway API can be split into different responsibility roles

![Hallo](images/resource-model.png)

### Reference 

  * https://gateway-api.sigs.k8s.io/
