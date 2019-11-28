**RBAC:**

This contains instructions to illustrate RBAC. RBAC is used to control access to cluster/namespace for users, groups and service accounts. We will create 1 role to be able to read pod detail from "default" namespace. We will create another role to be able to create/delete pods from "default" namespace.

**Pre-requisite for role creation:**

User should have permission to create roles:

    kubectl create clusterrolebinding cluster-admin-binding --clusterrole cluster-admin --user <userid>

**Create service account:**

First, create 2 separate service account that will be used to illustrate this.

    kubectl create serviceaccount read-account
    kubectl create serviceaccount owner-account

  **Create role:**
  
Create roles for reading and writing and bind them to the service account using rolebinding.

kubectl create -f readrole.yaml
kubectl create -f completerole.yaml

**Bind role to service account:**

Bind the above created role to the service account.

    kubectl create -f readrolebinding.yaml
    kubectl create -f completerolebinding.yaml

**Check roles:**

    kubectl --as=system:serviceaccount:default:read-account apply -f simple-pod.yaml
 This will fail as the read-account does not have permission to create pods.
    
    kubectl --as=system:serviceaccount:default:owner-account apply -f simple-pod.yaml
This will work as the owner-account has permission to create pods.
   

    kubectl --as=system:serviceaccount:default:read-account get pods

This will work since read-account has read pod access



