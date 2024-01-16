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



| TASK               | IP ADDRESS           |
| -------------------| ---------------------|
| `master-1`         | 192.168.2.104        |
| `master-2`         | 192.168.2.105        |
| `master-3`         | 192.168.2.106        |
| `Load Balancer`    | 192.168.2.185        |
| `worker-1`         | 192.168.2.107        |
| `worker-2`         | 192.168.2.108        |
| `worker-3`         | 192.168.2.109        |
