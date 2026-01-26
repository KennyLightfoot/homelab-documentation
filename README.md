# Homelab Documentation

> **Cloud Engineering & DevOps Skills Portfolio**  
> Hands-on infrastructure projects demonstrating real-world IT and cloud engineering capabilities

## 👨‍💻 About This Repository

This repository documents my homelab environment where I build, test, and learn enterprise IT infrastructure and DevOps practices. Each project includes detailed setup instructions, network diagrams, and lessons learned.

**Current Focus Areas:**
- Cloud Architecture (AWS, Azure)
- Container Orchestration (Docker, Kubernetes)
- Infrastructure as Code (Terraform, Ansible)
- CI/CD Pipelines
- Monitoring & Observability
- Network Security & Segmentation

---

## 🏗️ Infrastructure Overview

**Hardware:**
- Custom PC (Intel 14th Gen, 32GB RAM, 2TB NVMe)
- Virtualization: VMware Workstation Pro
- Storage: Dedicated 1TB SSD for lab environments

**Network:**
- Segmented VLANs for production/lab isolation
- pfSense virtual router (deployed)
- VLAN 10 (Production): 192.168.10.0/24
- VLAN 20 (Homelab): 192.168.20.0/24

---

## 📁 Projects

### ✅ [Project 1: Monitoring Infrastructure with Uptime Kuma](./projects/01-uptime-kuma/)
**Status:** Complete  
**Skills:** Docker, Container Management, Infrastructure Monitoring, Portainer

Set up enterprise-grade uptime monitoring for production websites using containerized applications. Includes automated alerts via Telegram and web-based dashboards.

**Tech Stack:** Ubuntu Server 24.04, Docker, Portainer, Uptime Kuma

---

### ✅ [Project 2: Network-Wide Ad Blocking with Pi-hole](./projects/02-pihole/)
**Status:** Complete  
**Skills:** DNS Management, Network Security, Docker, Ad Blocking

Deployed a containerized Pi-hole DNS server providing network-wide ad blocking, malicious domain filtering, and DNS query analytics. Built on dedicated Ubuntu Server VM with Docker, demonstrating service isolation and persistent data management.

**Tech Stack:** Ubuntu Server 24.04, Docker, Pi-hole v6.3

### ✅ Project 3: VLAN Network Segmentation
**Status:** Complete  
**Skills:** pfSense, Network Segmentation, Firewall Rules, Inter-VLAN Routing, Advanced Troubleshooting

Implemented enterprise-grade network isolation using pfSense virtual router with controlled access between Production and Homelab VLANs.

**Tech Stack:** pfSense 2.8.1, VMware Workstation Pro, Ubuntu Server 24.04

[View Project Details](./projects/03-vlan-segmentation/)

### 🚧 Project 4: SIEM with Wazuh
**Status:** Planned  
**Skills:** Security Monitoring, Log Analysis

### 🚧 Project 5: Infrastructure as Code with Terraform
**Status:** Planned  
**Skills:** Cloud Automation, AWS, IaC

### 🚧 Project 6: CI/CD Pipeline for Web Applications
**Status:** Planned  
**Skills:** GitHub Actions, Automated Deployment

### 🚧 Project 7: Active Directory Lab Environment
**Status:** Planned  
**Skills:** Windows Server, Domain Services, Group Policy

---

## 📚 Learning Resources

Resources and documentation that have been valuable in building this lab:
- [LearnLinuxTV YouTube Channel](https://www.youtube.com/c/LearnLinuxTV)
- [TechWorld with Nana](https://www.youtube.com/c/TechWorldwithNana)
- [AWS Documentation](https://docs.aws.amazon.com/)
- [Docker Documentation](https://docs.docker.com/)

---

## 📫 Connect

Building this lab as part of my journey into Cloud Engineering and DevOps. Always open to feedback, suggestions, and connecting with others in the field.

**Portfolio:** https://houstonmobilenotarypros.com/ https://www.proxrecon.com/
**LinkedIn:** https://www.linkedin.com/in/kenneth-lightfoot/
**WGU Program:** Cloud and Network Engineering - AWS  (In Progress)

---

*Last Updated: January 25, 2026*
