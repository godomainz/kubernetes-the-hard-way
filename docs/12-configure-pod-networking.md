# Provisioning Pod Network

We chose to use CNI - [weave](https://www.weave.works/docs/net/latest/kubernetes/kube-addon/) as our networking option.

### Install CNI plugins

Download the CNI Plugins required for weave on each of the worker nodes - `worker-1` and `worker-2`
This is the NEW way
wget https://github.com/containernetworking/plugins/releases/download/v0.8.7/cni-plugins-linux-amd64-v0.8.7.tgz

This is the OLD way
`wget https://github.com/containernetworking/plugins/releases/download/v0.7.5/cni-plugins-amd64-v0.7.5.tgz`

Extract it to /opt/cni/bin directory

This is the NEW way
sudo tar -xzvf cni-plugins-linux-amd64-v0.8.7.tgz  --directory /opt/cni/bin/


`sudo tar -xzvf cni-plugins-amd64-v0.7.5.tgz  --directory /opt/cni/bin/`

Reference: https://kubernetes.io/docs/concepts/extend-kubernetes/compute-storage-net/network-plugins/#cni

### Deploy Weave Network

Deploy weave network. Run only once on the `master` node.
{
    sudo ufw allow from any to any port 6783 proto tcp
    sudo ufw allow from any to any port 6783 proto udp
    sudo ufw allow from any to any port 6784 proto udp
}

{
    sudo ufw allow 6783/tcp
    sudo ufw allow 6783/udp
    sudo ufw allow 6784/udp
}

http://www.vassox.com/infrastructure/networking/enable-ip-forwarding-routing-on-ubuntu-18-04/
https://openvpn.net/faq/what-is-and-how-do-i-enable-ip-forwarding-on-linux/
https://askubuntu.com/questions/1050816/ubuntu-18-04-as-a-router
sudo sysctl net.ipv4.ip_forward=1 (do this command on all nodes)
echo "net.ipv4.ip_forward=1" | sudo tee -a /etc/sysctl.conf
sudo sysctl -p

https://www.weave.works/docs/net/latest/kubernetes/
sudo curl -L git.io/weave -o /usr/local/bin/weave
sudo chmod a+x /usr/local/bin/weave
weave setup
weave launch 192.168.111.136

`kubectl apply -f "https://cloud.weave.works/k8s/net?k8s-version=$(kubectl version | base64 | tr -d '\n')"`

Weave uses POD CIDR of `10.32.0.0/12` by default.

IF you want to change the POD CIDR use this command
kubectl apply -f "https://cloud.weave.works/k8s/net?k8s-version=$(kubectl version | base64 | tr -d '\n')&env.IPALLOC_RANGE=10.32.0.0/16"

## Verification

List the registered Kubernetes nodes from the master node:

```
master-1$ kubectl get pods -n kube-system -o wide --watch
```

> output

```
NAME              READY   STATUS    RESTARTS   AGE
weave-net-58j2j   2/2     Running   0          89s
weave-net-rr5dk   2/2     Running   0          89s
```

Reference: https://kubernetes.io/docs/tasks/administer-cluster/network-policy-provider/weave-network-policy/#install-the-weave-net-addon

Next: [Kube API Server to Kubelet Connectivity](13-kube-apiserver-to-kubelet.md)
