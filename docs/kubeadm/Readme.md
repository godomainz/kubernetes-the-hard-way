{
    LOADBALANCER=192.168.111.139:6443
    INTERNAL_IP=$(ip addr show ens33 | grep "inet " | awk '{print $2}' | cut -d / -f 1)
    echo $INTERNAL_IP
    sudo kubeadm init --control-plane-endpoint $LOADBALANCER --pod-network-cidr 10.244.0.0/16 --apiserver-advertise-address=$INTERNAL_IP --upload-certs
}


kubectl apply -f "https://cloud.weave.works/k8s/net?k8s-version=$(kubectl version | base64 | tr -d '\n')"


Your Kubernetes control-plane has initialized successfully!

To start using your cluster, you need to run the following as a regular user:

  mkdir -p $HOME/.kube
  sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
  sudo chown $(id -u):$(id -g) $HOME/.kube/config

You should now deploy a pod network to the cluster.
Run "kubectl apply -f [podnetwork].yaml" with one of the options listed at:
  https://kubernetes.io/docs/concepts/cluster-administration/addons/

You can now join any number of the control-plane node running the following command on each as root:

  sudo kubeadm join 192.168.111.139:6443 --token 69qp24.clnym1qhajedt47y \
    --discovery-token-ca-cert-hash sha256:92e21233379a1b7fa91cf77a7e63d272e6cfe91b06f1b8af14feda621b6daebb \
    --control-plane --certificate-key f2e0b41e6a0bebf2b24788aef6a3b48b8d919cb46a92c56b1e04328040b29380

Please note that the certificate-key gives access to cluster sensitive data, keep it secret!
As a safeguard, uploaded-certs will be deleted in two hours; If necessary, you can use
"kubeadm init phase upload-certs --upload-certs" to reload certs afterward.

Then you can join any number of worker nodes by running the following on each as root:

sudo kubeadm join 192.168.111.139:6443 --token 69qp24.clnym1qhajedt47y \
    --discovery-token-ca-cert-hash sha256:92e21233379a1b7fa91cf77a7e63d272e6cfe91b06f1b8af14feda621b6daebb