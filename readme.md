# 🛡️ Automated SOC Lab

## 🎯 Project Goal

Building a fully automated Security Operations Center (SOC) environment using **Infrastructure as Code (IaC)** principles. The goal is to simulate enterprise-grade attacks and detection pipelines using Splunk, Docker, and Sysmon.

## 🛠️ Tech Stack & Tools

- **Virtualization:** VMware Workstation Pro
- **Orchestration:** Vagrant
- **Containerization:** Docker
- **SIEM:** Splunk Enterprise
- **OS:** Ubuntu Server 22.04 (LTS) & Windows 10 Enterprise

## 🚀 Key Engineering Features

- **Reusability:** The entire lab can be destroyed and rebuilt with a single `vagrant up` command.
- **Networking:** Custom Port Forwarding to bypass NAT/Routing issues on Host OS.
- **Automation:** Bash provisioning scripts to auto-install Docker and deploy Splunk containers.
- **Resource Management:** Tuned memory allocation (4GB+) to prevent OOM (Out of Memory) errors for SIEM.

## 🔧 How to run

1. Install Vagrant & VMware Workstation.
2. Clone this repo.
3. Run `vagrant up`.
4. Access Splunk at `http://localhost:8000`.

## 🔧 Troubleshooting

- **Issue: Splunk is not responding / Container is down**
  - If the container didn't start correctly, try re-provisioning the machine (this re-runs the installation scripts): `vagrant provision`

- Issue: Network Timeout
  - If localhost:8000 is unreachable, ensure the port forwarding is active by reloading the network config: `vagrant reload`

- Manual Debugging Connect to the machine to check Docker logs:

```
vagrant ssh
sudo docker ps -a
sudo docker logs splunk
```
