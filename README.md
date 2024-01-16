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


```
curl -sfL https://get.rke2.io --output install.sh
chmod +x install.sh
sudo INSTALL_RKE2_TYPE=server ./install.sh
```

vim /etc/rancher/rke2/config.yaml
```
write-kubeconfig-mode: "0644"
tls-san:
  - devopsvoice.click
  - 192.168.2.85
  - 192.168.2.104
  - 192.168.2.105
  - 192.168.2.106
```