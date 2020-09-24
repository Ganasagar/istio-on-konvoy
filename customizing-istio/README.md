Istio is currently going through an a major migration from Helm --> Operator based instllation options. There are two kinds of customizations that you can apply to istio. 
1. Global level changes applied to the entire cluster 
2. Component level changes which are mostly relavent to the control-plane.


```bash
.....
.....
.....
        - name: istio # Istio is currently in Preview
          enabled: true
          values: |
            istioOperator:
              # Inspite of name, factor resource for entire control plane  
              components:
                pilot:
                  k8s:
                    hpaSpec:
                      minReplicas: 2
                    resources:
                      requests:
                        memory: 3072M
                        cpu: 100m
                egressGateways:
                - name: istio-egressgateway
                  enabled: true
              # These are the helm variables
              values:
                global: 
                # resource for envoy-proxies 
                  proxy:
                    resources:
                      requests:
                        cpu: 100m
                        memory: 128M
                      limits:
                        cpu: 2000m
                        memory: 1024M
                  proxy_init:
                    resources:
                      limits:
                        cpu: 100m
                        memory: 50M
                      requests:
                        cpu: 10m
                        memory: 10M
                
...
...
...
```