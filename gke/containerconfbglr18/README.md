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
Create cluster:

    make create-cluster
The above command creates 4 node cluster with necessary permissions. Network control policy is also enabled in the cluster.
Create namespace:

    make create-ns

**Nodeport, Load balancer and Ingress**
bookshelf/bookshelf.yaml
To deploy the entire application:

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
**Network control policy**
GKE uses Calico to implement Network control policy.
I have created 2 policies:
networkpolicy/allow_reviews_from_product.yaml - This allows only "productpage" to talk to "reviews" service.
networkpolicy/reject_reviews_from_all.yaml - This blocks all services from talking to "reviews" service. 
