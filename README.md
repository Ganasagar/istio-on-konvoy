# Istio-on-mke
This repo walk through steps for installing Istio on Konvoy 
### Prerequisites

Cloud platforms provide a wealth of benefits for the organizations that use them.
There’s no denying, however, that adopting the cloud can put strains on DevOps teams.
Developers must use microservices to architect for portability, meanwhile operators are managing extremely large hybrid and multi-cloud deployments.
Istio lets you connect, secure, control, and observe services.

At a high level, Istio helps reduce the complexity of these deployments, and eases the strain on your development teams.
It is a completely open source service mesh that layers transparently onto existing distributed applications.
It is also a platform, including APIs that let it integrate into any logging platform, or telemetry or policy system.
Istio’s diverse feature set lets you successfully, and efficiently, run a distributed microservice architecture, and provides a uniform way to secure, connect, and monitor microservices.

D2iQ istio's implementation is based on the upstream Istio project. However it comes with curated list of addons like Jaeger-tracing, Grafana, Prometheus, Cert-Manager etc which are thorougly tested before deployment and are deelpy integrated with Konvoy clusters. 

Edit the `cluster.yaml` file to change the default value of Istio enabled from `false` to `true` in 1 corresponding field:
```
...
    - enabled: false
      name: istio
    - enabled: true
      name: kibana
    - enabled: true
...
    - enabled: true
      name: istio
    - enabled: true
      name: kibana
    - enabled: true
...
```

```bash
konvoy up --yes --upgrade --force-upgrade
```
You should see an the resulting output should contain istio as in the addons list towards the bottom
```
STAGE [Deploying Enabled Addons]
elasticsearch                                                          [OK]
gatekeeper                                                             [OK]
reloader                                                               [OK]
cert-manager                                                           [OK]
elasticsearchexporter                                                  [OK]
konvoyconfig                                                           [OK]
opsportal                                                              [OK]
external-dns                                                           [OK]
prometheus                                                             [OK]
dashboard                                                              [OK]
dex                                                                    [OK]
kommander                                                              [OK]
kudo                                                                   [OK]
traefik                                                                [OK]
fluentbit                                                              [OK]
kibana                                                                 [OK]
prometheusadapter                                                      [OK]
traefik-forward-auth                                                   [OK]
awsebscsiprovisioner                                                   [OK]
defaultstorageclass-protection                                         [OK]
dex-k8s-authenticator                                                  [OK]
velero                                                                 [OK]
kube-oidc-proxy                                                        [OK]
istio                                                                  [OK]
```




Currently the addon deployed using Konvoy only confirm the triggering of Istio Operator which is reponsible for deploying Istio in the Istio-namespace. To confirm that Istio is installed and is up and running, we will take a couple steps.


1. Check if the istio pods are running:

```bash
kubectl -n istio-system get pods
```

Output:
```
NAME                                    READY   STATUS      RESTARTS   AGE
istio-crd-1.6.4-27hye-dxjq9             0/1     Completed   0          49m
istio-ingressgateway-59ddcb9559-qtqfs   1/1     Running     0          57m
istio-ingressgateway-59ddcb9559-vkbtj   1/1     Running     0          57m
istio-operator-65458b9b8b-bwjf5         1/1     Running     0          29h
istio-tracing-dbfcd555-2jq89            1/1     Running     0          57m
istiod-79f4958ff6-t7wmn                 1/1     Running     0          57m
istiod-79f4958ff6-tg8hw                 1/1     Running     0          57m
kiali-6f9b78cdc6-grrfv                  1/1     Running     0          57m                           0/1     Completed   0          26m

```
2. If you do not see the pods up and running, then review the operator logs which is responsible for deploying Istio. Use the operator name from above output
```bash
kubectl logs <operator-pod-name>
```
Example:
```bash
kubectl logs istio-operator-65458b9b8b-bwjf5 
```
Output:
```bash
........
.........
.........
2020-09-22T22:16:24.138819Z	info	installer	Processing resources from manifest: EgressGateways for CR istio-default-istio-system-EgressGateways
2020-09-22T22:16:24.138835Z	info	installer	Processing resources from manifest: Telemetry for CR istio-default-istio-system-Telemetry
2020-09-22T22:16:24.138851Z	info	installer	Generated manifest objects are the same as cached for component EgressGateways.
2020-09-22T22:16:24.138861Z	info	installer	Generated manifest objects are the same as cached for component Telemetry.
2020-09-22T22:16:24.141229Z	info	installer	Generated manifest objects are the same as cached for component Cni.
2020-09-22T22:16:24.230793Z	info	installer	The following objects differ between generated manifest and cache:
 - ConfigMap:istio-system:kiali
2020-09-22T22:16:24.233005Z	info	installer	Generated manifest objects are the same as cached for component IngressGateways.
2020-09-22T22:16:24.233181Z	info	installer	updating resource: ConfigMap/istio-system/kiali
- Pruning removed resources
- Processing resources for Addons.
✔ Addons installed
```
Note: If you ever misconfigure the `cluster.yaml` before running `konvoy up` the operator might not be able to deploy Istio and you will be able to see errors related to syntax in the operator logs. 


### Download and install istioctl to interact with the mesh

Download the latest version of the Istioctl:
```bash
curl -sL https://istio.io/downloadIstioctl | sh -
```
Add the istioctl client to your path, on a macOS or Linux system:
```bash
export PATH=$PATH:$HOME/.istioctl/bin
```
You can get an overview of your mesh using the proxy-status command:
```bash
istioctl proxy-status
```
You should see output like below:
```bash
NAME                                                   CDS        LDS        EDS        RDS          PILOT                            VERSION
istio-ingressgateway-6db99c76dc-zqb2s.istio-system     SYNCED     SYNCED     SYNCED     NOT SENT     istio-pilot-7684976f67-vxgbc     1.6.8
```

