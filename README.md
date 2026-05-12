# 🛡️ Greenbone GVM: Resilient Deployment & Troubleshooting Case Study

A professional technical roadmap for deploying the **Greenbone Vulnerability Manager (GVM)** in a high-latency, resource-constrained VMware environment. This project documents the "Level 2" engineering solutions required to overcome packet fragmentation, TLS timeouts, and service port collisions.

---

## 🚀 Project Overview
This deployment utilizes the **Greenbone Community Edition** on Ubuntu 24.04. The primary objective was to build a stable vulnerability management stack that coexists with a **Wazuh XDR** manager. 

**Environment:** VMware Virtual Platform  
**Resources:** 4GB RAM / 2 vCPUs  

---

## 🛠️ Complete Technical Workflow & Command History

### 1. Environment Hardening (Network & Permissions)
Before deployment, the system was optimized to handle virtual network fragmentation and Docker permissions.
```bash
# Fix Docker permissions
sudo usermod -aG docker $USER && newgrp docker

# Resolve TLS Handshake Timeouts by adjusting MTU
sudo ip link set dev ens33 mtu 1400
```

### 2. Due to registry timeouts, heavy images were pulled individually using a persistence loop to ensure 100% data integrity.
Targeted pulls for heavy data/engine components
```bash
sudo docker pull registry.community.greenbone.net/community/scap-data:latest
sudo docker pull registry.community.greenbone.net/community/vulnerability-tests:latest
sudo docker pull registry.community.greenbone.net/community/ospd-openvas:stable
```
### 3. Port Mapping & XDR Coexistence
To prevent conflict with the Wazuh Dashboard (Port 443), the compose.yaml was re-engineered to route Greenbone through Port 4443.
```bash
# Configuration snippet from compose.yaml
nginx:
  ports:
    - 4443:443   # Remapped to coexist with Wazuh
    - 9392:9392
```
###  4. Infrastructure Launch & Monitoring
```bash
# Create directory and download original compose file
mkdir greenbone-lab && cd greenbone-lab
curl -f -L [https://greenbone.github.io/docs/latest/_static/compose.yaml](https://greenbone.github.io/docs/latest/_static/compose.yaml) -o compose.yaml
```
Project Summary: The Greenbone Resilient Deployment
This project successfully established a Greenbone Community Edition (GVM) vulnerability management stack within a complex virtualized environment. The journey involved moving beyond standard installation scripts to perform advanced system engineering and troubleshooting across the following layers:

Network Layer (L3): Resolved packet fragmentation and TLS handshake timeouts by manually tuning the MTU to 1400, ensuring stable communication between the VMware guest and the external Docker registry.

Infrastructure Layer: Engineered a serialized image acquisition strategy (the "Sniper" method) with an automated persistence loop to overcome high-latency network conditions.

Application Layer (L7): Resolved a critical port collision with the Wazuh Dashboard by re-engineering the Nginx reverse proxy to listen on Port 4443, allowing for the simultaneous operation of XDR and Vulnerability Management tools on a single host.
