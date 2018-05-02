This directory contains steps to create Cassandra cluster in GKE. This was tried with Kubernetes version 1.8.8.
This is based on example here:
https://kubernetes.io/docs/tutorials/stateful-application/cassandra/

Create cluster:
gcloud container clusters create cass-cluster --num-nodes=3  --machine-type=n1-standard-4

Create service and statefulset:
kubectl create -f cassandra-service.yaml
kubectl create -f cassandra-statefulset.yaml
