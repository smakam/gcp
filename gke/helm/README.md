**This directory contains Helm examples**

**Prerequisites:**

GKE cluster<
Install helm client and Tiller server component.

**Helm version after successful installation:**

$ helm version
Client: &version.Version{SemVer:"v2.8.2", GitCommit:"a80231648a1473929271764b920a8e346f6de844", GitTreeState:"clean"}
Server: &version.Version{SemVer:"v2.8.2", GitCommit:"a80231648a1473929271764b920a8e346f6de844", GitTreeState:"clean"}

**helloapp:**

This application is used to illustrate helm templates where nodeport value is passed as runtime argument.
This is a simple Go application that is implemented as a Container. There are 4 resources:
Pod
Deployment
Service
Ingress

Typically, node ports are dynamically picked by Kubernetes. We can statically specify 
node ports if needed.

**Usage:**

helm install ./helloapp
- This will use default values for nodeport

helm install --set mynodeport=30503 ./helloapp
- This will pass nodeport=30503 to service manifest using helm templates. 

**Delete**

helm delete \<releasename\>
