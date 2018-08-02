This file contains instructions for running through the Kubernetes networking demo. This was presented in Container conference, Bangalore. GKE is used here. 
All commands are listed in "**Makefile**".
There are 3 parts:

 - Multi-container pod 
 - Illustrating Nodeport, Load balancer and Ingress along with Network control policy with Bookshelf multi-container application
 - Illustrate Istio service mesh with Bookshelf multi-service application

 **Multi-container pod:**

     multicontpod/pod.yaml
There are 2 containers "nginx" and "alpine" with a shared volume. This example illustrates:

 - Shared network namespace
 - Shared volumes

```
    $ make get-mc
    
    kubectl get pods mc1 --namespace dev
    
    NAME  READY STATUS  RESTARTS AGE
    
    mc1 2/2 Running 0  2h

```
**Cluster creation:**

This example uses a single cluster with 2 namespaces "default" and "dev". "dev" namespace is used to illustrate load balancer and network control policy. "default" namespace is used to illustrate Istio service mesh.
*Create cluster:*

    make create-cluster
The above command creates 4 node cluster with necessary permissions. Network control policy is also enabled in the cluster.
Create namespace:

    make create-ns

**Nodeport, Load balancer and Ingress**

File: bookshelf/bookshelf.yaml

*To deploy the entire application:*

    make deploy-bsdev
This will create pods, deployments, services. For this example, "productpage" service is exposed using nodeport, load balancer and Ingress.
```
$ make get-stuff-dev

kubectl get pods --namespace dev && kubectl get svc --namespace dev && kubectl get ingress --namespace dev

NAME  READY STATUS  RESTARTS AGE

details-v1-64b86cd49-zp8bx  1/1 Running 0  2h

mc1 2/2 Running 0  2h

productpage-v1-84f77f8747-qpvqs 1/1 Running 0  2h

ratings-v1-5f46655b57-pwcht 1/1 Running 0  2h

reviews-v1-ff6bdb95b-hcxq6  1/1 Running 0  2h

reviews-v2-5799558d68-mvx4j 1/1 Running 0  2h

reviews-v3-58ff7d665b-fvmr4 1/1 Running 0  2h

NAME TYPE CLUSTER-IP EXTERNAL-IP  PORT(S)  AGE

details  ClusterIP  10.3.242.101 <none> 9080/TCP 2h

productpage  NodePort 10.3.243.200 <none> 9080:32024/TCP 2h

productpagenlb LoadBalancer 10.3.252.191 35.197.41.33 80:30524/TCP 2h

ratings  ClusterIP  10.3.248.129 <none> 9080/TCP 2h

reviews  ClusterIP  10.3.248.49  <none> 9080/TCP 2h

NAME  HOSTS  ADDRESS PORTS AGE

gateway mydomain.com 35.241.52.239 80  2h
```
Load balancer IP address is the Network load balancer endpoint in GCP. Ingress IP address is the GLB endpoint in GCP. 

**Network control policy**

GKE uses Calico to implement Network control policy.
I have created 2 policies:
networkpolicy/allow_reviews_from_product.yaml - This allows only "productpage" to talk to "reviews" service.
networkpolicy/reject_reviews_from_all.yaml - This blocks all services from talking to "reviews" service. 

*Block reviews from other services:*

    make rejectreviews
*Look at the rules:*
```
$ make getnetrules

kubectl get networkpolicies -n dev

NAME  POD-SELECTOR AGE

hello-reject-reviews-from-all app=reviews  4s
```
At this point, the reviews service will not be accessible.

*Delete rules:*

    make deletenetrules

**Istio service mesh:**
The first step is to deploy Istio in the cluster. This includes Istio control plane services like Pilot, mixer, Citadel and dataplane service Envoy.

    make deploy-istio
After deploying Istio, we can look at Istio services. Istio gets deployed in its own namespace.
```
$ make get-stuff-istio

kubectl get pods --namespace istio-system && kubectl get svc --namespace istio-system && kubectl get ingress --namespace istio-system

NAME  READY STATUS  RESTARTS AGE

grafana-89f97d9c-ttbw9  1/1 Running 0  2d

istio-ca-59f6dcb7d9-vvz7t 1/1 Running 0  2d

istio-ingress-779649ff5b-rxntw  1/1 Running 0  2d

istio-mixer-7f4fd7dff-bpgkc 3/3 Running 0  2d

istio-pilot-5f5f76ddc8-7ksr9  2/2 Running 0  2d

istio-sidecar-injector-7947777478-vl459 1/1 Running 0  2d

jaeger-deployment-559c8b9b8-8zt4x 1/1 Running 0  2d

prometheus-cf8456855-jf795  1/1 Running 0  2d

servicegraph-59ff5dbbff-7rqzm 1/1 Running 0  2d

NAME TYPE CLUSTER-IP EXTERNAL-IP  PORT(S)  AGE

grafana  ClusterIP  10.3.241.239 <none> 3000/TCP 2d

istio-ingress  LoadBalancer 10.3.254.62  35.227.187.171 80:30535/TCP,443:31134/TCP 2d

istio-mixer  ClusterIP  10.3.243.28  <none> 9091/TCP,15004/TCP,9093/TCP,9094/TCP,9102/TCP,9125/UDP,42422/TCP 2d

istio-pilot  ClusterIP  10.3.245.142 <none> 15003/TCP,8080/TCP,9093/TCP,443/TCP  2d

istio-sidecar-injector ClusterIP  10.3.248.56  <none> 443/TCP  2d

jaeger-agent ClusterIP  None <none> 5775/UDP,6831/UDP,6832/UDP 2d

jaeger-collector ClusterIP  10.3.247.97  <none> 14267/TCP,14268/TCP,9411/TCP 2d

jaeger-query LoadBalancer 10.3.240.87  35.233.203.120 80:31781/TCP 2d

prometheus ClusterIP  10.3.241.125 <none> 9090/TCP 2d

servicegraph ClusterIP  10.3.240.231 <none> 8088/TCP 2d

zipkin ClusterIP  None <none> 9411/TCP 2d

No resources found.
```
*Monitor services:*
Istio provides nice integration with third party monitoring, tracing and visualization services. For this application, we have deployed Grafana for monitoring, Jaeger for tracing, dotviz for service visualization.
*Start monitoring:*
make start-monitoring-services

*Access monitoring:*
http://localhost:3000
Access visualization:
http://localhost:8088/dotviz
Access tracing:
http://http://localhost:16686/

*Deploy the bookshelf application:*

    make deploy-bs
When application is deployed on top of Istio, the proxy gets created for each application pod. 

*Using Istio rules:*
Initially, the reviews service will get load balanced between the 3 versions.
Lets make a change to use only version v1:

    make v1
Lets make change to have only user "jason" access v2 and remaining users access v1:

    make v2jason
List rules:

    make getistiorules
Delete rules:

    make deleteistiorules
Load balance 50% of traffic to v3 and remaining to v1:

    make v350
Do a rolling upgrade to v3:

    make v3

**Delete services and cluster:**

    make delete-bs
    make delete-bsdev
    make delete-cluster

