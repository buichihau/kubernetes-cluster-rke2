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


## Info Nodes

| TASK               | IP ADDRESS           |  HOSTNAME            |
| -------------------| ---------------------| ---------------------|
| `master-1`         | 192.168.2.104        | master1.rke2.com     |  
| `master-2`         | 192.168.2.105        | master2.rke2.com     |
| `master-3`         | 192.168.2.106        | master3.rke2.com     |
| `load balancer`    | 192.168.2.185        | lb.rke2.com          |
| `worker-1`         | 192.168.2.107        | worker1.rke2.com     |
| `worker-2`         | 192.168.2.108        | worker2.rke2.com     |
| `worker-3`         | 192.168.2.109        | worker3.rke2.com     | 


#  Set up the First Server Node (Master Node)
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
* start the service
```
sudo systemctl start rke2-server
sudo systemctl enable rke2-server
```


