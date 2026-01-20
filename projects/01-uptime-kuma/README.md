# Project 1: Monitoring Infrastructure with Uptime Kuma

![Status](https://img.shields.io/badge/Status-Complete-success)
![Difficulty](https://img.shields.io/badge/Difficulty-Beginner-green)

## 📋 Project Overview

Deployed a containerized uptime monitoring solution to track availability and performance of production websites. This project demonstrates container management, infrastructure monitoring, and alert configuration.

## 🎯 Learning Objectives

- Install and configure Docker on Ubuntu Server
- Deploy and manage containers using Portainer
- Configure uptime monitoring for multiple websites
- Set up automated alerting via Telegram
- Understand container networking and port mapping

## 🛠️ Technologies Used

| Technology | Version | Purpose |
|------------|---------|---------|
| Ubuntu Server | 24.04 LTS | Host OS |
| Docker | 27.x | Container runtime |
| Portainer | CE 2.x | Container management UI |
| Uptime Kuma | 1.x | Uptime monitoring |
| VMware Workstation | Pro 25H2 | Virtualization platform |

## 🏗️ Architecture
```
┌─────────────────────────────────────────┐
│          Windows 11 Host                │
│  (Intel 14th Gen, 32GB RAM)             │
│                                         │
│  ┌───────────────────────────────────┐ │
│  │   VMware Workstation Pro          │ │
│  │                                   │ │
│  │  ┌─────────────────────────────┐ │ │
│  │  │  Ubuntu Server 24.04 VM     │ │ │
│  │  │  (4GB RAM, 2 vCPU, 50GB)    │ │ │
│  │  │                             │ │ │
│  │  │  ┌──────────────────────┐   │ │ │
│  │  │  │  Docker Engine       │   │ │ │
│  │  │  │                      │   │ │ │
│  │  │  │  ┌────────────────┐  │   │ │ │
│  │  │  │  │   Portainer    │  │   │ │ │
│  │  │  │  │   Port: 9443   │  │   │ │ │
│  │  │  │  └────────────────┘  │   │ │ │
│  │  │  │                      │   │ │ │
│  │  │  │  ┌────────────────┐  │   │ │ │
│  │  │  │  │  Uptime Kuma   │  │   │ │ │
│  │  │  │  │   Port: 3001   │  │   │ │ │
│  │  │  │  └────────────────┘  │   │ │ │
│  │  │  └──────────────────────┘   │ │ │
│  │  │                             │ │ │
│  │  │  IP: 192.168.226.128        │ │ │
│  │  └─────────────────────────────┘ │ │
│  └───────────────────────────────────┘ │
└─────────────────────────────────────────┘
```

## 📝 Implementation Steps

### 1. Environment Setup

**Prepared VMware Host:**
- Formatted 1TB SSD for VM storage
- Configured VMware to use D:\VMs as default location

**Created Ubuntu Server VM:**
```bash
# VM Specifications
- OS: Ubuntu Server 24.04 LTS
- RAM: 4GB
- CPU: 2 cores
- Disk: 50GB
- Network: NAT
```

### 2. Ubuntu Server Installation

- Selected standard Ubuntu Server installation
- Configured network via DHCP (VMware NAT)
- Enabled OpenSSH server for remote access
- Created administrative user account

### 3. Docker Installation
```bash
# Update system packages
sudo apt update && sudo apt upgrade -y

# Install Docker using official script
curl -fsSL https://get.docker.com -o get-docker.sh
sudo sh get-docker.sh

# Add user to docker group
sudo usermod -aG docker $USER

# Verify installation
docker --version
docker ps
```

### 4. Portainer Deployment
```bash
# Create persistent volume
docker volume create portainer_data

# Deploy Portainer
docker run -d \
  -p 9000:9000 \
  -p 9443:9443 \
  --name=portainer \
  --restart=always \
  -v /var/run/docker.sock:/var/run/docker.sock \
  -v portainer_data:/data \
  portainer/portainer-ce:latest
```

**Access:** https://192.168.226.128:9443

### 5. Uptime Kuma Deployment

**Deployed via Portainer UI:**
- Container name: `uptime-kuma`
- Image: `louislam/uptime-kuma:1`
- Port mapping: `3001:3001`
- Volume: `/home/big-dog/uptime-kuma-data:/app/data`
- Restart policy: Always

**Access:** http://192.168.226.128:3001

### 6. Monitor Configuration

**Added monitors for:**
1. Houston Mobile Notary Pros (https://houstonmobilenotarypros.com)
2. ProxRecon (https://www.proxrecon.com)

**Settings:**
- Check interval: 60 seconds
- Retries before alert: 3
- Notifications: Telegram bot integration

## 📊 Results

**Monitoring Metrics:**
- ✅ 100% uptime visibility
- ✅ Response time tracking (avg ~350-400ms)
- ✅ SSL certificate expiration monitoring
- ✅ Real-time Telegram alerts
- ✅ Historical uptime data

## 💡 Key Learnings

1. **Container Advantages:** Containers provide isolated, portable environments that are easy to deploy and manage
2. **Volume Persistence:** Data volumes ensure configurations survive container restarts
3. **Port Mapping:** Understanding host-to-container port mapping is critical for service accessibility
4. **Restart Policies:** Always-restart ensures services survive server reboots
5. **Web-based Management:** Portainer significantly simplifies Docker management vs CLI-only

## 🚀 Next Steps

- [ ] Add more monitors for additional services
- [ ] Configure email notifications as backup to Telegram
- [ ] Set up status page for public uptime visibility
- [ ] Implement log aggregation for troubleshooting
- [ ] Add resource monitoring for the VM itself

## 🔗 Resources

- [Docker Documentation](https://docs.docker.com/)
- [Portainer Documentation](https://docs.portainer.io/)
- [Uptime Kuma GitHub](https://github.com/louislam/uptime-kuma)
- [Ubuntu Server Guide](https://ubuntu.com/server/docs)

---

**Project Completion Date:** January 20, 2026  
**Time Investment:** ~3 hours  
**Difficulty Rating:** Beginner-friendly
