sudo apt-get install bash-completion -y
source /usr/share/bash-completion/bash_completion
echo 'source <(kubectl completion bash)' >>~/.bashrc
exec bash

kubectl apply -f metric-server.yaml

kubectl get apiservice v1beta1.metrics.k8s.io -o yaml


kubectl apply -f https://raw.githubusercontent.com/kubernetes/dashboard/master/aio/deploy/recommended.yaml
kubectl create serviceaccount dashboard-admin-sa
kubectl create clusterrolebinding dashboard-admin-sa --clusterrole=cluster-admin --serviceaccount=default:dashboard-admin-sa
kubectl get secrets
kubectl describe secret dashboard-admin-sa-token-kw7vn
kubectl proxy
http://localhost:8001/api/v1/namespaces/kubernetes-dashboard/services/https:kubernetes-dashboard:/proxy/




http://192.168.11.252:8001/api/v1/namespaces/kube-system/services/https:kubernetes-dashboard:/proxy/

kubectl delete -f https://raw.githubusercontent.com/kubernetes/dashboard/master/aio/deploy/recommended.yaml
kubectl delete -f https://github.com/kubernetes-sigs/metrics-server/releases/download/v0.3. 7/components.yaml