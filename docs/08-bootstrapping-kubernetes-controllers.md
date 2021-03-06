# Bootstrapping the Kubernetes Control Plane

In this lab you will bootstrap the Kubernetes control plane across 2 compute instances and configure it for high availability. You will also create an external load balancer that exposes the Kubernetes API Servers to remote clients. The following components will be installed on each node: Kubernetes API Server, Scheduler, and Controller Manager.

## Prerequisites

The commands in this lab must be run on each controller instance: `master-1`, and `master-2`. Login to each controller instance using SSH Terminal. Example:

### Running commands in parallel with tmux

[tmux](https://github.com/tmux/tmux/wiki) can be used to run commands on multiple compute instances at the same time. See the [Running commands in parallel with tmux](01-prerequisites.md#running-commands-in-parallel-with-tmux) section in the Prerequisites lab.

## Provision the Kubernetes Control Plane

Create the Kubernetes configuration directory:

```
sudo mkdir -p /etc/kubernetes/config
```

### Download and Install the Kubernetes Controller Binaries

Download the official Kubernetes release binaries:

```
This is the NEW way

{
  cd ~
  mkdir -p kubernetes_config
  cd kubernetes_config
  wget -q --show-progress --https-only --timestamping https://github.com/kubernetes/kubernetes/releases/download/v1.18.8/kubernetes.tar.gz
  tar -xvf kubernetes.tar.gz
  echo Y |  ./kubernetes/cluster/get-kube-binaries.sh
  tar -xvf kubernetes/server/kubernetes-server-linux-amd64.tar.gz
}




This is the OLD way

wget -q --show-progress --https-only --timestamping \
  "https://storage.googleapis.com/kubernetes-release/release/v1.13.0/bin/linux/amd64/kube-apiserver" \
  "https://storage.googleapis.com/kubernetes-release/release/v1.13.0/bin/linux/amd64/kube-controller-manager" \
  "https://storage.googleapis.com/kubernetes-release/release/v1.13.0/bin/linux/amd64/kube-scheduler" \
  "https://storage.googleapis.com/kubernetes-release/release/v1.13.0/bin/linux/amd64/kubectl"
```

Reference: https://kubernetes.io/docs/setup/release/#server-binaries

Install the Kubernetes binaries:

```
This is the NEW way
{
  chmod +x kubernetes/server/bin/kube-apiserver kubernetes/server/bin/kube-controller-manager kubernetes/server/bin/kube-scheduler kubernetes/server/bin/kubectl
  sudo mv kubernetes/server/bin/kube-apiserver kubernetes/server/bin/kube-controller-manager kubernetes/server/bin/kube-scheduler kubernetes/server/bin/kubectl /usr/local/bin/
}


This is the OLD way
{
  chmod +x kube-apiserver kube-controller-manager kube-scheduler kubectl
  sudo mv kube-apiserver kube-controller-manager kube-scheduler kubectl /usr/local/bin/
}
```

### Configure the Kubernetes API Server

```
{
  sudo mkdir -p /var/lib/kubernetes/

  sudo cp ca.crt ca.key kube-apiserver.crt kube-apiserver.key kube-proxy.crt kube-proxy.key \
    service-account.key service-account.crt \
    etcd-server.key etcd-server.crt \
    encryption-config.yaml /var/lib/kubernetes/
}
```

The instance internal IP address will be used to advertise the API Server to members of the cluster. Retrieve the internal IP address for the current compute instance:

```
INTERNAL_IP=$(ip addr show ens33 | grep "inet " | awk '{print $2}' | cut -d / -f 1)
```

Verify it is set

```
echo $INTERNAL_IP
```

Create the `kube-apiserver.service` systemd unit file:
IMPORTANT DO NOT PUT SPACES AFTER \\
```


cat <<EOF | sudo tee /etc/systemd/system/kube-apiserver.service
[Unit]
Description=Kubernetes API Server
Documentation=https://github.com/kubernetes/kubernetes

[Service]
ExecStart=/usr/local/bin/kube-apiserver \\
  --advertise-address=${INTERNAL_IP} \\
  --apiserver-count=3 \\
  --audit-log-maxage=30 \\
  --audit-log-maxbackup=3 \\
  --audit-log-maxsize=100 \\
  --audit-log-path=/var/log/audit.log \\
  --authorization-mode=Node,RBAC \\
  --bind-address=0.0.0.0 \\
  --client-ca-file=/var/lib/kubernetes/ca.crt \\
  --enable-admission-plugins=NodeRestriction,ServiceAccount,PodPreset \\
  --enable-swagger-ui=true \\
  --enable-bootstrap-token-auth=true \\
  --etcd-cafile=/var/lib/kubernetes/ca.crt \\
  --etcd-certfile=/var/lib/kubernetes/etcd-server.crt \\
  --etcd-keyfile=/var/lib/kubernetes/etcd-server.key \\
  --etcd-servers=https://192.168.111.134:2379,https://192.168.111.135:2379 \\
  --event-ttl=1h \\
  --encryption-provider-config=/var/lib/kubernetes/encryption-config.yaml \\
  --kubelet-certificate-authority=/var/lib/kubernetes/ca.crt \\
  --kubelet-client-certificate=/var/lib/kubernetes/kube-apiserver.crt \\
  --kubelet-client-key=/var/lib/kubernetes/kube-apiserver.key \\
  --kubelet-preferred-address-types=InternalIP,ExternalIP,Hostname \\
  --requestheader-client-ca-file=/var/lib/kubernetes/kube-proxy.crt \\
  --requestheader-allowed-names=front-proxy-client \\
  --requestheader-extra-headers-prefix=X-Remote-Extra- \\
  --requestheader-group-headers=X-Remote-Group \\
  --requestheader-username-headers=X-Remote-User \\
  --proxy-client-cert-file=/var/lib/kubernetes/kube-proxy.crt \\
  --proxy-client-key-file=/var/lib/kubernetes/kube-proxy.key \\
  --enable-aggregator-routing=true \\
  --kubelet-https=true \\
  --runtime-config=api/all=true \\
  --service-account-key-file=/var/lib/kubernetes/service-account.crt \\
  --service-cluster-ip-range=10.96.0.0/24 \\
  --service-node-port-range=30000-32767 \\
  --tls-cert-file=/var/lib/kubernetes/kube-apiserver.crt \\
  --tls-private-key-file=/var/lib/kubernetes/kube-apiserver.key \\
  --runtime-config=settings.k8s.io/v1alpha1=true \\
  --allow-privileged=true \\
  --v=2
Restart=on-failure
RestartSec=5

[Install]
WantedBy=multi-user.target
EOF
```

### Configure the Kubernetes Controller Manager

Copy the `kube-controller-manager` kubeconfig into place:

```
sudo cp kube-controller-manager.kubeconfig /var/lib/kubernetes/
```

Create the `kube-controller-manager.service` systemd unit file:

```
cat <<EOF | sudo tee /etc/systemd/system/kube-controller-manager.service
[Unit]
Description=Kubernetes Controller Manager
Documentation=https://github.com/kubernetes/kubernetes

[Service]
ExecStart=/usr/local/bin/kube-controller-manager \\
  --address=0.0.0.0 \\
  --cluster-cidr=192.168.111.0/24 \\
  --cluster-name=kubernetes \\
  --cluster-signing-cert-file=/var/lib/kubernetes/ca.crt \\
  --cluster-signing-key-file=/var/lib/kubernetes/ca.key \\
  --kubeconfig=/var/lib/kubernetes/kube-controller-manager.kubeconfig \\
  --leader-elect=true \\
  --root-ca-file=/var/lib/kubernetes/ca.crt \\
  --service-account-private-key-file=/var/lib/kubernetes/service-account.key \\
  --service-cluster-ip-range=10.96.0.0/24 \\
  --use-service-account-credentials=true \\
  --v=2
Restart=on-failure
RestartSec=5

[Install]
WantedBy=multi-user.target
EOF
```

### Configure the Kubernetes Scheduler

Copy the `kube-scheduler` kubeconfig into place:

```
sudo cp kube-scheduler.kubeconfig /var/lib/kubernetes/
```

Create the `kube-scheduler.service` systemd unit file:

```
cat <<EOF | sudo tee /etc/systemd/system/kube-scheduler.service
[Unit]
Description=Kubernetes Scheduler
Documentation=https://github.com/kubernetes/kubernetes

[Service]
ExecStart=/usr/local/bin/kube-scheduler \\
  --kubeconfig=/var/lib/kubernetes/kube-scheduler.kubeconfig \\
  --address=127.0.0.1 \\
  --leader-elect=true \\
  --v=2
Restart=on-failure
RestartSec=5

[Install]
WantedBy=multi-user.target
EOF
```

### Start the Controller Services

```
{
  sudo systemctl daemon-reload
  sudo systemctl enable kube-apiserver kube-controller-manager kube-scheduler
  sudo systemctl start kube-apiserver kube-controller-manager kube-scheduler
}
```

> Allow up to 10 seconds for the Kubernetes API Server to fully initialize.


### Verification

```
kubectl get componentstatuses --kubeconfig admin.kubeconfig
```

```
NAME                 STATUS    MESSAGE              ERROR
controller-manager   Healthy   ok
scheduler            Healthy   ok
etcd-0               Healthy   {"health": "true"}
etcd-1               Healthy   {"health": "true"}
```

> Remember to run the above commands on each controller node: `master-1`, and `master-2`.

## The Kubernetes Frontend Load Balancer

In this section you will provision an external load balancer to front the Kubernetes API Servers. The `kubernetes-the-hard-way` static IP address will be attached to the resulting load balancer.


### Provision a Network Load Balancer

Login to `loadbalancer` instance using SSH Terminal.

#Install nginx
sudo apt update
sudo apt install nginx -y
sudo ufw enable
sudo ufw app list
sudo ufw allow 'Nginx HTTP'
sudo ufw allow 'Nginx HTTPS'
systemctl status nginx

openssl genrsa -out ssl.key 2048
openssl req -new -key ssl.key -subj "/CN=ssl" -out ssl.csr
openssl x509 -req -in ssl.csr -signkey ssl.key -CAcreateserial  -out ssl.crt -days 1000

create docker-compose.yml
version: "3"
services:
  nginxproxy:
    image: nginx
    restart: always
    ports:
     - 6443:80
    volumes:
     - "./nginx.conf:/etc/nginx/nginx.conf"
     - "./ssl.crt:/etc/nginx/certs/ssl.crt"
     - "./ssl.key:/etc/nginx/certs/ssl.key"

---------------------------------------------------------------
cat > docker-compose.yml <<EOF
version: "3"
services:
  nginxproxy:
    image: nginx
    restart: always
    ports:
     - 6443:80
    volumes:
     - "./nginx.conf:/etc/nginx/nginx.conf"
EOF
---------------------------------------------------------------

create nginx.conf
events {
  multi_accept on;
  worker_connections  4096;
}
http {
	upstream myproject {
            least_conn;
	    server 192.168.111.246:6443;
	    server 192.168.111.247:6443;
	}
	server {
            listen 6443 ssl;
            ssl_certificate     /home/akila/nginx/ca.crt;
    	    ssl_certificate_key /home/akila/nginx/ca.key;
            ssl_protocols       TLSv1 TLSv1.1 TLSv1.2;
	   
	    location / {
               proxy_redirect          off;
               proxy_http_version      1.1;
               proxy_set_header Host   $host;
               proxy_set_header        X-Real-IP       $remote_addr;
               proxy_set_header        X-Forwarded-For $proxy_add_x_forwarded_for;
               proxy_set_header        Upgrade $http_upgrade;
               proxy_set_header        Connection "upgrade";
	       proxy_pass              https://myproject;
	    }
	}
}


------------------------------------------------------------------------

nginx.conf
events {
  multi_accept on;
  worker_connections  4096;
}
http {
	upstream myproject {
            least_conn;
	    server 192.168.111.246:6443;
	    server 192.168.111.247:6443;
	}
	server {
            listen 6443 ssl;
            ssl_certificate     /home/akila/nginx/ca.crt;
    	      ssl_certificate_key /home/akila/nginx/ca.key;
            ssl_protocols       TLSv1 TLSv1.1 TLSv1.2;
	   
	    location / {
               proxy_redirect          off;
               proxy_http_version      1.1;
               proxy_set_header Host   $host;
               proxy_set_header        X-Real-IP       $remote_addr;
               proxy_set_header        X-Forwarded-For $proxy_add_x_forwarded_for;
               proxy_set_header        Upgrade $http_upgrade;
               proxy_set_header        Connection "upgrade";
	       proxy_pass              https://myproject;
	    }
	}
}


------------------------------------------------------------------------

If you don't have docker installed use above config file to configure nginx

```
#Install HAProxy
loadbalancer# sudo apt-get update && sudo apt-get install -y haproxy

```
------------------------------------------------------------
loadbalancer# cat <<EOF | sudo tee /etc/haproxy/haproxy.cfg 
frontend kubernetes
    bind 192.168.111.245:6443
----------------------------------------------------------

```
loadbalancer# cat <<EOF | sudo tee haproxy.cfg 
frontend kubernetes
    bind 0.0.0.0:6443
    option tcplog
    mode tcp
    default_backend kubernetes-master-nodes

backend kubernetes-master-nodes
    mode tcp
    balance roundrobin
    option tcp-check
    server master1 192.168.111.134:6443 check fall 3 rise 2
    server master2 192.168.111.135:6443 check fall 3 rise 2
EOF
```

```
loadbalancer# sudo service haproxy restart
```


---------------------------------------------------------------
cat > docker-compose.yml <<EOF
version: "3"
services:
  haproxy:
    image: haproxy
    restart: always
    ports:
     - 6443:6443
    volumes:
     - "./haproxy.cfg:/usr/local/etc/haproxy/haproxy.cfg"
EOF
---------------------------------------------------------------

### Verification

Make a HTTP request for the Kubernetes version info:

```
curl  https://192.168.111.133:6443/version -k
```

> output

```
{
  "major": "1",
  "minor": "13",
  "gitVersion": "v1.13.0",
  "gitCommit": "ddf47ac13c1a9483ea035a79cd7c10005ff21a6d",
  "gitTreeState": "clean",
  "buildDate": "2018-12-03T20:56:12Z",
  "goVersion": "go1.11.2",
  "compiler": "gc",
  "platform": "linux/amd64"
}
```

Next: [Bootstrapping the Kubernetes Worker Nodes](09-bootstrapping-kubernetes-workers.md)
