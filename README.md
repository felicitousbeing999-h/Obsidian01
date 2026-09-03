# 🧠 DevOps, Cloud Engineering & Core Infrastructure Vault

<div align="center">

![Platform](https://img.shields.io/badge/Platform-Multi--Cloud-informational?style=flat-square&color=2E86AB&logo=kubernetes&logoColor=white)
![Engine](https://img.shields.io/badge/Engine-Container%20Orchestration-success?style=flat-square&color=A23B72&logo=docker&logoColor=white)
![IaC](https://img.shields.io/badge/IaC-Infrastructure%20as%20Code-critical?style=flat-square&color=F18F01&logo=terraform&logoColor=white)

</div>

---

## 🎯 Welcome to My Engineering "Second Brain"

This is **not** a textbook. This is a living, breathing vault of **production-grade patterns**, **real-world diagnostics**, and **battle-tested runbooks** built from hands-on infrastructure engineering. Every line of documentation comes from solving actual problems in distributed systems.

Think of it as your **diagnostic compass** for navigating the chaos of modern cloud infrastructure. ✨

---

## 🗺️ Core Infrastructure Domains

<table>
<tr>
<td align="center" width="33%">

### ☸️ **Container Orchestration**
<img src="https://img.shields.io/badge/Kubernetes-Mastery-blue?style=for-the-badge&logo=kubernetes" alt="K8s"/>

- Pod troubleshooting frameworks
- Cluster scheduling optimization
- Multi-tier microservices architecture
- [K8s Kodekloud Labs](./K8s-Kodekloud)

</td>
<td align="center" width="33%">

### ☁️ **Cloud Architecture**
<img src="https://img.shields.io/badge/Multi Cloud-AWS%20%7C%20Azure%20%7C%20GCP-orange?style=for-the-badge" alt="Cloud"/>

- Azure/AWS administration
- Managed infrastructure patterns
- IaC automation frameworks
- [Azure DevOps](./Azure-Devops) & [Azure Guides](./Azure)

</td>
<td align="center" width="33%">

### 🔒 **Systems & Security**
<img src="https://img.shields.io/badge/Security-Hardening%20%26%20Observability-critical?style=for-the-badge&logo=security" alt="Security"/>

- Network observability & logging
- Identity and access management
- Host security configurations
- [Security Runbooks](./sec)

</td>
</tr>
</table>

---

## 📚 Quick Navigation to Key Resources

### 🔥 **Mission-Critical Runbooks**

<div style="display: grid; grid-template-columns: repeat(auto-fit, minmax(300px, 1fr)); gap: 20px;">

#### 📝 **Version Control & Git Workflows**
```
├── Git - Quick Decision Cheat Sheet
│   └── 7-second engineering decision matrix
│       • rebase master • push --force-with-lease
│       • stash • revert • cherry-pick patterns
│
└── Git Stash & Git Rebase Guide  
    └── Linear history maintenance
        • Clean in-flight modifications
        • Parallel feature streams
```
[📖 Browse Git Resources](./kloud-devops)

---

#### 🐳 **High-Availability Microservices**
```
├── Nginx + PHP-FPM Pod RCA Runbook
│   └── Structural root-cause analysis
│       • Inter-container networking
│       • Filesystem permissions debugging
│       • Socket mapping configuration
│
└── Kubernetes Deployment Troubleshooting
    └── Active failure mitigation matrix
        • CrashLoopBackOff resolution
        • Image registry error diagnosis
        • Liveness/readiness probe tuning
```
[📖 Troubleshooting Guide](./Troubleshoot%20Deployment%20issues%20in%20Kubernetes.md)

---

#### 🏗️ **Cloud Architecture & Strategy**
```
├── Azure Systems Administration Blueprint
│   └── Enterprise architectural patterns
│       • Storage topology optimization
│       • Cloud governance frameworks
│       • Multi-tier infrastructure design
│
└── Google Cloud Associate Engineer (ACE) Study Guide
    └── Comprehensive certification prep
        • GCP service architecture
        • Scaling & performance patterns
```
[📖 View Study Materials](./Google%20Cloud%20Associate%20Cloud%20Engineer%20(ACE)%20Comprehensive%20Study%20Guide.md)

</div>

---

## 🎓 Learning Paths & Lab Roadmap

| Status | Milestone | Coverage |
|:------:|-----------|----------|
| ✅ | **Advanced Version Control** | Zero-loss cherry-pick strategies • Secure history rewriting |
| ✅ | **Multi-Container Pod Optimization** | PHP-FPM/Nginx runtime isolation • Network boundaries |
| 🚧 | **Declarative GitOps Pipelines** | Jenkins multi-stage automation • Ansible orchestration |
| 🚀 | **Immutable Infrastructure** | AWS Terraform multi-tier baselines • State management |

---

## 🛠️ The Complete Toolkit

```yaml
Orchestration:
  Engines: 🐋 Kubernetes (K8s) | Docker Engine | Docker Compose
  Scale: Multi-node clusters | High-availability patterns

Cloud Platforms:
  Public: ☁️ Amazon Web Services (AWS)
  Enterprise: 🟦 Microsoft Azure  
  Google: 🔵 Google Cloud Platform (GCP)

Automation & IaC:
  Infrastructure: 🏗️ Terraform | CloudFormation
  Configuration: 📋 Ansible Playbooks | Bash Scripting
  GitOps: 🔄 Declarative pipeline automation

Core Systems:
  Operating System: 🐧 GNU/Linux Administration
  Networking: 🔗 Network protocols | DNS | TCP/IP
  Web Servers: ⚙️ Nginx Proxy | PHP-FPM | Load Balancing
```

---

## 📂 Repository Structure

```
Obsidian01/
├── 📁 kloud-devops/              # Version control & DevOps patterns
├── 📁 Azure/                     # Azure cloud architecture guides
├── 📁 Azure-Devops/              # Azure DevOps platform guides
├── 📁 K8s-Kodekloud/             # Kubernetes hands-on labs
├── 📁 Manage scale with GKE/     # Google Kubernetes Engine scaling
├── 📁 linux-admin/               # GNU/Linux administration
├── 📁 observabillity/            # Logging, monitoring & telemetry
├── 📁 sec/                       # Security hardening & best practices
├── 📁 data/                      # Reference data & configurations
│
├── 📄 Hands-On KitOps Demo...    # AI artifact OCI packaging
├── 📄 Google Cloud ACE Guide     # Comprehensive study guide
├── 📄 DevSecOpsMic-final-canvas  # Visual topology diagrams
├── 📄 Welcome.md                 # Quick start guide
└── 🎨 SVG Diagrams               # Visual architecture references
```

---

## 🌟 Featured Deep Dives

### 🎯 Latest Explorations

<details>
<summary><b>📊 Container & Microservices Patterns</b></summary>

- [Hands-On KitOps Demo - OCI Packaging for AI Artifacts](./Hands-On%20KitOps%20Demo%20OCI%20Packaging%20for%20AI%20Artifact.md)
- Multi-container networking strategies
- Persistent volume configuration
- Service mesh integration patterns

</details>

<details>
<summary><b>🎓 Cloud Certification Paths</b></summary>

- [Google Cloud Associate Cloud Engineer (ACE) Comprehensive Study Guide](./Google%20Cloud%20Associate%20Cloud%20Engineer%20(ACE)%20Comprehensive%20Study%20Guide.md)
- AZ-104 Azure Administrator prerequisites
- Hands-on lab environments
- Real-world scenario walkthroughs

</details>

<details>
<summary><b>🔧 Operational Runbooks</b></summary>

- [Kubernetes Deployment Troubleshooting Matrix](./Troubleshoot%20Deployment%20issues%20in%20Kubernetes.md)
- Git workflow decision matrices
- Nginx + PHP-FPM stack diagnostics
- Container runtime debugging

</details>

---

## 💡 Philosophy Behind This Vault

> **"Infrastructure engineering is about preventing fires, not just fighting them."**

Every note here represents:
- ✅ **Tested Solutions** — From sandbox labs to production deployments
- ✅ **Root-Cause Analysis** — Not just band-aids, but systemic fixes
- ✅ **Decision Frameworks** — Quick reference matrices for high-pressure scenarios
- ✅ **Continuous Learning** — Updated as technologies evolve

---

## 🚀 Quick Start

1. **New to Kubernetes?** → Head to [K8s-Kodekloud](./K8s-Kodekloud)
2. **Azure Cloud?** → Explore [Azure Guides](./Azure) & [Azure-Devops](./Azure-Devops)
3. **Troubleshooting containers?** → Read [Kubernetes Deployment Issues](./Troubleshoot%20Deployment%20issues%20in%20Kubernetes.md)
4. **Git workflow issues?** → Use the [Git Decision Cheat Sheet](./kloud-devops)
5. **Studying for GCP?** → Review [Google Cloud ACE Guide](./Google%20Cloud%20Associate%20Cloud%20Engineer%20(ACE)%20Comprehensive%20Study%20Guide.md)

---

## 📊 Visual Architecture References

- 🗺️ **[Azure Systems Blueprint](./az-104-administrator-prerequisites.svg)** — Enterprise topology mapping
- 🎨 **[DevSecOps Architecture Canvas](./DevSecOpsMic-final-canvas.canvas)** — Security-first design patterns
- 📸 **[Supporting Diagrams & Screenshots](./Pasted%20image%2020260729002747.png)** — Visual topology references

---

## 🔗 Connected to Production

This vault mirrors **real-world workspace documentation standards**. All architectural structures, diagnostic procedures, and configuration runbooks are:
- ✅ Version controlled with Git
- ✅ Continuously updated from live systems
- ✅ Built for teams managing production infrastructure
- ✅ Validated through hands-on labs

---

## 📝 Contributing to Your Own Vault

> *"Your documentation should evolve as quickly as your infrastructure does."*

Key practices implemented in this vault:
- Daily documentation of engineering challenges
- Markdown-first approach for readability
- Obsidian integration for cross-linking
- Git history for tracking evolution

---

<div align="center">

### 🎓 Keep Learning • Stay Sharp • Build Reliable Infrastructure

**Last Updated:** September 2026 | [View All Files](https://github.com/felicitousbeing999-h/Obsidian01) | [Report Issues](https://github.com/felicitousbeing999-h/Obsidian01/issues)

</div>
