
make sure matric server is running --> https://github.com/kubernetes-sigs/metrics-server
kubectl apply -f .
kubectl -n kubernetes-dashboard describe secret $(kubectl -n kubernetes-dashboard get secret | grep admin-user | awk '{print $1}')

kubectl proxy
kubectl proxy --address='0.0.0.0' --port=8001 --accept-hosts='.*'
kubectl proxy --address='0.0.0.0' --port=8001 --accept-hosts='^*$'
kubectl proxy --address="192.168.111.134" -p 8001 --accept-hosts='^*$'
kubectl proxy --address='0.0.0.0' --port=8001 --accept-hosts='^*$' --kubeconfig=admin.kubeconfig

http://localhost:8001/api/v1/namespaces/kubernetes-dashboard/services/https:kubernetes-dashboard:/proxy/

