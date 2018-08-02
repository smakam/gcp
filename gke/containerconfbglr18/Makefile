PROJECT_ID=$(shell gcloud config list project --format=flattened | awk 'FNR == 1 {print $$2}')
ZONE=us-west1-b
CLUSTER_NAME=my-istio-cluster
ZIPKIN_POD_NAME=$(shell kubectl -n istio-system get pod -l app=zipkin -o jsonpath='{.items[0].metadata.name}')
JAEGER_POD_NAME=$(shell kubectl -n istio-system get pod -l app=jaeger -o jsonpath='{.items[0].metadata.name}')
SERVICEGRAPH_POD_NAME=$(shell kubectl -n istio-system get pod -l app=servicegraph -o jsonpath='{.items[0].metadata.name}')
GRAFANA_POD_NAME=$(shell kubectl -n istio-system get pod -l app=grafana -o jsonpath='{.items[0].metadata.name}')
PROMETHEUS_POD_NAME=$(shell kubectl -n istio-system get pod -l app=prometheus -o jsonpath='{.items[0].metadata.name}')
GCLOUD_USER=$(shell gcloud config get-value core/account)
EXTERNAL_IP=35.227.187.171

create-cluster:
	gcloud container --project "$(PROJECT_ID)" clusters create "$(CLUSTER_NAME)" --zone "$(ZONE)" --machine-type "n1-standard-1" --image-type "COS" --enable-network-policy --disk-size "100" --scopes "https://www.googleapis.com/auth/compute","https://www.googleapis.com/auth/devstorage.read_only","https://www.googleapis.com/auth/logging.write","https://www.googleapis.com/auth/monitoring","https://www.googleapis.com/auth/servicecontrol","https://www.googleapis.com/auth/service.management.readonly","https://www.googleapis.com/auth/trace.append" --num-nodes "4" --network "default" --enable-cloud-logging --enable-cloud-monitoring --cluster-version=1.9
	kubectl create clusterrolebinding cluster-admin-binding --clusterrole=cluster-admin --user=$(GCLOUD_USER)
deploy-istio:
	kubectl apply -f istio-0.6/install/kubernetes/istio.yaml
	./istio-0.6/install/kubernetes/webhook-create-signed-cert.sh --service istio-sidecar-injector --namespace istio-system --secret sidecar-injector-certs
	kubectl apply -f istio-0.6/install/kubernetes/istio-sidecar-injector-configmap-release.yaml
	cat istio-0.6/install/kubernetes/istio-sidecar-injector.yaml | ./istio-0.6/install/kubernetes/webhook-patch-ca-bundle.sh > istio-0.6/install/kubernetes/istio-sidecar-injector-with-ca-bundle.yaml
	kubectl apply -f istio-0.6/install/kubernetes/istio-sidecar-injector-with-ca-bundle.yaml
	kubectl apply -f istio-0.6/install/kubernetes/addons/prometheus.yaml
	kubectl apply -n istio-system -f istio-0.6/install/kubernetes/addons/jaeger.yaml
	kubectl apply -f istio-0.6/install/kubernetes/addons/grafana.yaml
	kubectl apply -f istio-0.6/install/kubernetes/addons/servicegraph.yaml
	kubectl label namespace default istio-injection=enabled
create-ns:
	-kubectl create ns dev

# Multi-container pod example
create-mc:
	kubectl apply -f multicontpod/pod.yaml --namespace dev
delete-mc:
	kubectl delete -f multicontpod/pod.yaml --namespace dev
get-mc:
	kubectl get pods mc1 --namespace dev 
desc-mc:
	kubectl describe pods mc1 --namespace dev 
check-volume:
	kubectl exec mc1 --namespace dev -c alpine -- /bin/cat /html/index.html 
	kubectl exec mc1 --namespace dev -c web -- /bin/cat /usr/share/nginx/html/index.html
access-web:
	kubectl exec mc1 --namespace dev -c alpine -- wget -qO- localhost 
web-sh:
	kubectl exec mc1 --namespace dev -c web -i -t -- bash 
alpine-sh:
	kubectl exec mc1 --namespace dev -c alpine -i -t -- sh


# Dev ns is used to test network control policy. default ns is used to test Istio.
# Network control policy and Istio can be applied to same ns as well.
# dev ns
deploy-bsdev:
	kubectl apply --namespace=dev -f bookshelf/bookshelf.yaml
get-stuff-dev:
	kubectl get pods --namespace dev && kubectl get svc --namespace dev && kubectl get ingress --namespace dev
# network policy
getnetrules:
	kubectl get networkpolicies -n dev
allowreviews:
	kubectl apply -f networkpolicy/allow_reviews_from_product.yaml -n dev
rejectreviews:
	kubectl apply -f networkpolicy/reject_reviews_from_all.yaml -n dev
deletenetrules:
	-kubectl delete -f networkpolicy/allow_reviews_from_product.yaml -n dev
	-kubectl delete -f networkpolicy/reject_reviews_from_all.yaml -n dev

# default ns for Istio
deploy-bs:
	kubectl apply -f bookshelf/bookshelf-istio.yaml
get-stuff:
	kubectl get pods && kubectl get svc && kubectl get ingress
get-stuff-istio:
	kubectl get pods --namespace istio-system && kubectl get svc --namespace istio-system && kubectl get ingress --namespace istio-system

#Istio rules
getistiorules:
	istioctl get routerules -o yaml
#Direct all traffic to v1 version
v1:
	istioctl create -f ./bookshelf/istio-rules/route-rule-all-v1.yaml
#Create v2 rule for user “jason”
v2jason:
	istioctl create -f ./bookshelf/istio-rules/route-rule-reviews-test-v2.yaml
#Upgrade 50% to v3
v350:
	istioctl create -f ./bookshelf/istio-rules/route-rule-reviews-50-v3.yaml
#Transfer 100% to v3
v3:
	istioctl replace -f ./bookshelf/istio-rules/route-rule-reviews-v3.yaml
sendtraffic:
	for ((i=1;i<=100;i++)); do curl -o /dev/null -s -w "%{http_code}\n" http://$(EXTERNAL_IP)/productpage; done
#Delete all rules
deleteistiorules:
	-istioctl delete -f ./bookshelf/istio-rules/route-rule-all-v1.yaml 
	-istioctl delete -f ./bookshelf/istio-rules/route-rule-reviews-test-v2.yaml
	-istioctl delete -f ./bookshelf/istio-rules/route-rule-reviews-50-v3.yaml 
	-istioctl delete -f ./bookshelf/istio-rules/route-rule-reviews-v3.yaml

start-monitoring-services:
	$(shell kubectl -n istio-system port-forward $(JAEGER_POD_NAME) 16686:16686 & kubectl -n istio-system port-forward $(SERVICEGRAPH_POD_NAME) 8088:8088 & kubectl -n istio-system port-forward $(GRAFANA_POD_NAME) 3000:3000 & kubectl -n istio-system port-forward $(PROMETHEUS_POD_NAME) 9090:9090))

# Delete resources
delete-bsdev:
	kubectl delete --namespace=dev -f bookshelf/bookshelf.yaml
delete-bs:
	kubectl delete -f bookshelf/bookshelf-istio.yaml
delete-cluster:
	gcloud container clusters delete "$(CLUSTER_NAME)" --zone "$(ZONE)"
