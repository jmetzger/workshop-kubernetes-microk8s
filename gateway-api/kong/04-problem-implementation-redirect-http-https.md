# Problems implentation

## It should be like this (gateway api - spec)

  * https://gateway-api.sigs.k8s.io/guides/http-redirect-rewrite/#http-to-https-redirects

## But in kong it needs to be done like this (and only works like this) 

  * https://docs.konghq.com/kubernetes-ingress-controller/latest/guides/services/https-redirect/

## The reason: Redirect is not yet implemented in Kong 

  * https://github.com/kubernetes-sigs/gateway-api/blob/main/conformance/reports/v1.0.0/kong-kubernetes-ingress-controller/v3.1.1-report.yaml
