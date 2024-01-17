# Deploy HA Kubernetes Cluster on Linux using RKE2
**https://docs.rke2.io/architecture**

## System Requirements

- OS: Ubuntu focal 20.04 or Centos 7, 8
- Platform: On-prem/Cloud/Virtualization
- RAM: 4GB Minimum (we recommend at least 8GB)
- CPU: 2 Minimum (we recommend at least 4CPU)
- NIC: 1 NIC for node
- Firewall: Disabled
- SELinux: Disabled
- SWAP : Disabled


## Step 1 – Set up Linux 8 Nodes

| TASK               | IP ADDRESS           |  HOSTNAME            |
| -------------------| ---------------------| ---------------------|
| `master-1`         | 192.168.2.104        | master1.rke2.com     |  
| `master-2`         | 192.168.2.105        | master2.rke2.com     |
| `master-3`         | 192.168.2.106        | master3.rke2.com     |
| `load balancer`    | 192.168.2.85         | lb.rke2.com          |
| `worker-1`         | 192.168.2.107        | worker1.rke2.com     |
| `worker-2`         | 192.168.2.108        | worker2.rke2.com     |
| `worker-3`         | 192.168.2.109        | worker3.rke2.com     | 

* Set the hostnames as shown
```
##On Node1
sudo hostnamectl set-hostname master1.rke2.com 

##On Node2
sudo hostnamectl set-hostname master2.rke2.com 

##On Node3
sudo hostnamectl set-hostname master3.rke2.com 

##On Loadbalancer(Node4)
sudo hostnamectl set-hostname lb.rke2.com 

##On Node5
sudo hostnamectl set-hostname worker1.rke2.com

##On Node6
sudo hostnamectl set-hostname worker2.rke2.com

##On Node7
sudo hostnamectl set-hostname worker3.rke2.com
```

* Add the hostnames to /etc/hosts on each node
```
192.168.2.104 master1.rke2.com
192.168.2.105 master2.rke2.com
192.168.2.106 master3.rke2.com
192.168.2.107 worker1.rke2.com
192.168.2.108 worker2.rke2.com
192.168.2.109 worker3.rke2.com
192.168.2.85 lb.rke2.com
```

* Disabled SELinux
```
setenforce 0
sed -i --follow-symlinks 's/^SELINUX=enforcing/SELINUX=disabled/' /etc/sysconfig/selinux
```

* Disabled Firewall
```
systemctl disable firewalld >/dev/null 2>&1
systemctl stop firewalld
```

* Disabled swap
```
sed -i '/swap/d' /etc/fstab
swapoff -a
```

# Step 2 – Load Balancer Deployment
- Load balancer can be of the following types:
  - Nginx
  - HAProxy
  - Existing Load Balancer

* Use HAProxy

#  Step 3 – Set up the First Server Node (Master Node)
* Install RKE2 server
```
curl -sfL https://get.rke2.io --output install.sh
chmod +x install.sh
sudo INSTALL_RKE2_TYPE=server ./install.sh
```

* RKE2 config file for first server
```
cat >>/etc/rancher/rke2/config.yaml<<EOF
token: rke2-k8s-sTill-win-@-zay
tls-san:
  - lb.rke2.com
  - 192.168.2.85
  - 192.168.2.104
  - 192.168.2.105
  - 192.168.2.106
  - master1.rke2.com 
  - master2.rke2.com
  - master3.rke2.com 
node-ip: 192.168.2.104
node-name: master1.rke2.com
EOF
```
* Start the service
```
sudo systemctl start rke2-server
sudo systemctl enable rke2-server
```

* install kubeclt
```
curl -LO https://dl.k8s.io/release/`curl -LS https://dl.k8s.io/release/stable.txt`/bin/linux/amd64/kubectl
chmod +x ./kubectl
sudo mv ./kubectl /usr/local/bin/kubectl
export KUBECONFIG=/etc/rancher/rke2/rke2.yaml
kubectl version
```

* After some time, check if the node and pods are up:
```
$ kubectl get nodes
NAME               STATUS   ROLES                       AGE   VERSION
master1.rke2.com   Ready    control-plane,etcd,master   9h    v1.26.12+rke2r1

```

# Step 4 – Set up additional Server Nodes (Master Nodes)

* Install RKE2 server
```
curl -sfL https://get.rke2.io --output install.sh
chmod +x install.sh
sudo INSTALL_RKE2_TYPE=server ./install.sh
```

* RKE2 config file for second server (master2)
```
cat >>/etc/rancher/rke2/config.yaml<<EOF
server: https://192.168.2.85:9345
token: rke2-k8s-sTill-win-@-zay
tls-san:
  - lb.rke2.com
  - 192.168.2.85
  - 192.168.2.104
  - 192.168.2.105
  - 192.168.2.106
  - master1.rke2.com 
  - master2.rke2.com
  - master3.rke2.com 
node-ip: 192.168.2.105
node-name: master2.rke2.com
EOF
```

* RKE2 config file for third  server (master3)
```
cat >>/etc/rancher/rke2/config.yaml<<EOF
server: https://192.168.2.85:9345
token: rke2-k8s-sTill-win-@-zay
tls-san:
  - lb.rke2.com
  - 192.168.2.85
  - 192.168.2.104
  - 192.168.2.105
  - 192.168.2.106
  - master1.rke2.com 
  - master2.rke2.com
  - master3.rke2.com 
node-ip: 192.168.2.106
node-name: master3.rke2.com
EOF
```

* start the service
```
sudo systemctl start rke2-server
sudo systemctl enable rke2-server
```
* After some time, check the status of the nodes
```
kubectl get node -owide
NAME               STATUS   ROLES                       AGE   VERSION           INTERNAL-IP     EXTERNAL-IP   OS-IMAGE                KERNEL-VERSION                CONTAINER-RUNTIME
master1.rke2.com   Ready    control-plane,etcd,master   10h   v1.26.12+rke2r1   192.168.2.104   <none>        CentOS Linux 7 (Core)   3.10.0-1160.90.1.el7.x86_64   containerd://1.7.11-k3s2
master2.rke2.com   Ready    control-plane,etcd,master   22m   v1.26.12+rke2r1   192.168.2.105   <none>        CentOS Linux 7 (Core)   3.10.0-1160.90.1.el7.x86_64   containerd://1.7.11-k3s2
master3.rke2.com   Ready    control-plane,etcd,master   11m   v1.26.12+rke2r1   192.168.2.106   <none>        CentOS Linux 7 (Core)   3.10.0-1160.90.1.el7.x86_64   containerd://1.7.11-k3s2
```

# Step 5 – Set up Agent Nodes (Worker Nodes)

* Install RKE2 agent
```
curl -sfL https://get.rke2.io --output install.sh
chmod +x install.sh
sudo INSTALL_RKE2_TYPE=agent ./install.sh
```

* RKE2 config file for Agent Nodes (Worker 1)
```
cat >>/etc/rancher/rke2/config.yaml<<EOF
server: https://192.168.2.85:9345
token: rke2-k8s-sTill-win-@-zay
tls-san:
  - lb.rke2.com
  - 192.168.2.85
  - 192.168.2.104
  - 192.168.2.105
  - 192.168.2.106
  - master1.rke2.com 
  - master2.rke2.com
  - master3.rke2.com 
node-ip: 192.168.2.107
node-name: worker1.rke2.com
EOF
```

* RKE2 config file for Agent Nodes (Worker 2)
```
cat >>/etc/rancher/rke2/config.yaml<<EOF
server: https://192.168.2.85:9345
token: rke2-k8s-sTill-win-@-zay
tls-san:
  - lb.rke2.com
  - 192.168.2.85
  - 192.168.2.104
  - 192.168.2.105
  - 192.168.2.106
  - master1.rke2.com 
  - master2.rke2.com
  - master3.rke2.com 
node-ip: 192.168.2.108
node-name: worker2.rke2.com
EOF
```

* RKE2 config file for Agent Nodes (Worker 3)
```
cat >>/etc/rancher/rke2/config.yaml<<EOF
server: https://192.168.2.85:9345
token: rke2-k8s-sTill-win-@-zay
tls-san:
  - lb.rke2.com
  - 192.168.2.85
  - 192.168.2.104
  - 192.168.2.105
  - 192.168.2.106
  - master1.rke2.com 
  - master2.rke2.com
  - master3.rke2.com 
node-ip: 192.168.2.109
node-name: worker3.rke2.com
EOF
```

* start the service rke2-agent
```
sudo systemctl start rke2-agent
sudo systemctl enable rke2-agent
```

* After some time, check the status of the nodes
```
 kubectl get node -owide
NAME               STATUS   ROLES                       AGE    VERSION           INTERNAL-IP     EXTERNAL-IP   OS-IMAGE                KERNEL-VERSION                CONTAINER-RUNTIME
master1.rke2.com   Ready    control-plane,etcd,master   12h    v1.26.12+rke2r1   192.168.2.104   <none>        CentOS Linux 7 (Core)   3.10.0-1160.90.1.el7.x86_64   containerd://1.7.11-k3s2
master2.rke2.com   Ready    control-plane,etcd,master   130m   v1.26.12+rke2r1   192.168.2.105   <none>        CentOS Linux 7 (Core)   3.10.0-1160.90.1.el7.x86_64   containerd://1.7.11-k3s2
master3.rke2.com   Ready    control-plane,etcd,master   119m   v1.26.12+rke2r1   192.168.2.106   <none>        CentOS Linux 7 (Core)   3.10.0-1160.90.1.el7.x86_64   containerd://1.7.11-k3s2
worker1.rke2.com   Ready    <none>                      23m    v1.26.12+rke2r1   192.168.2.107    <none>        CentOS Linux 7 (Core)   3.10.0-1160.90.1.el7.x86_64   containerd://1.7.11-k3s2
worker2.rke2.com   Ready    <none>                      23m    v1.26.12+rke2r1   192.168.2.108    <none>        CentOS Linux 7 (Core)   3.10.0-1160.90.1.el7.x86_64   containerd://1.7.11-k3s2
worker3.rke2.com   Ready    <none>                      23m    v1.26.12+rke2r1   192.168.2.109    <none>        CentOS Linux 7 (Core)   3.10.0-1160.90.1.el7.x86_64   containerd://1.7.11-k3s2
```
