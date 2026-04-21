# 🖥️ Container Monitoring Stack

A production-style monitoring stack built with **Prometheus**, **Grafana**, **Alertmanager**, and a suite of exporters — all orchestrated with Docker Compose. Designed for observability across containers, hosts, web servers, and network endpoints.

---

## 🧩 Stack Components

| Service | Port | Description |
|---------|------|-------------|
| **Prometheus** | `9090` | Scrapes and stores metrics from all exporters |
| **Grafana** | `3000` | Visualization dashboards |
| **Alertmanager** | `9093` | Handles alerts and sends email notifications via Gmail SMTP |
| **Node Exporter** | `9100` | Host-level metrics (CPU, RAM, disk, network) |
| **NGINX** | `80` | Sample monitored web service |
| **NGINX Exporter** | `9113` | Exposes NGINX stub_status metrics for Prometheus |
| **Apache (httpd)** | `8085` | Sample Apache web server with virtual hosts |
| **Apache Exporter** | `9117` | Exposes Apache mod_status metrics for Prometheus |
| **cAdvisor** | `8080` | Per-container resource & performance metrics |
| **Blackbox Exporter** | `9115` | Probes HTTP, HTTPS, ICMP, SSH, TCP endpoints |

---

## ✨ Key Features

- **Host Monitoring** — CPU, memory, disk, and network via Node Exporter (local + remote VMs + EC2)
- **Container Observability** — Per-container CPU/RAM/network via cAdvisor
- **Web Server Metrics** — Live NGINX and Apache traffic, connections, and request rates
- **Blackbox Probing** — Endpoint availability checks across multiple protocols:
  - **ICMP** — Ping probes for hosts and gateways
  - **HTTP/HTTPS** — Response code validation for internal sites and external URLs
  - **SSH** — Banner checks on remote servers
  - **TCP/LDAPS** — Port connectivity checks for FreeIPA (DNS, Kerberos, LDAP, LDAPS)
- **Email Alerting** — Gmail SMTP alerts via Alertmanager with pre-defined rules
- **Grafana Dashboards** — 9 pre-built dashboards covering the full stack
- **Wazuh Integration** — Grafana dashboard connected to OpenSearch/Wazuh for security events
- **Ansible Automation** — Node Exporter deployment automated with Ansible roles

---

## 📊 Grafana Dashboards

| Dashboard | Description |
|-----------|-------------|
| **Node Exporter Full** | Host-level metrics (CPU, memory, disk, network) |
| **cAdvisor** | Per-container resource usage |
| **NGINX Exporter** | NGINX connection and request rates |
| **Apache Dashboard** | Apache request rates and worker status |
| **Blackbox Exporter** | HTTP/HTTPS probe results and latency |
| **ICMP Dashboard** | Ping probe results and round-trip times |
| **SSH Dashboard** | SSH banner probe status |
| **FreeIPA Dashboard** | FreeIPA service port connectivity |
| **Wazuh / OpenSearch** | Security events and alerts |

All dashboards are auto-provisioned on stack startup from `grafana/dashboards/`.

---

## 🚀 Getting Started

### Prerequisites
- Docker & Docker Compose
- Git

### 1. Clone the repository

```bash
git clone https://github.com/your-username/monitor-containers.git
cd monitor-containers
```

### 2. Configure secrets


make `.env` with your actual values:

```env
SMTP_AUTH_PASSWORD=your_gmail_app_password
SMTP_TO=alert_recipient@gmail.com
SMTP_FROM=your_sender@gmail.com
OPENSEARCH_PASSWORD=your_opensearch_password
```

> **Note:** To generate a Gmail App Password, go to  
> Google Account → Security → 2-Step Verification → App Passwords

### 3. Configure Alertmanager

Edit `alertmanager.yml` with your SMTP credentials:

```yaml
auth_username: 'your_sender@gmail.com'
auth_password: "your_gmail_app_password"
to: 'alert_recipient@gmail.com'
from: 'your_sender@gmail.com'
```

### 4. Start the stack

```bash
docker compose up -d
```

### 5. Access the dashboards

| Service | URL |
|---------|-----|
| Grafana | http://localhost:3000 |
| Prometheus | http://localhost:9090 |
| Alertmanager | http://localhost:9093 |
| cAdvisor | http://localhost:8080 |
| Apache | http://localhost:8085 |
| Blackbox Exporter | http://localhost:9115 |

> Default Grafana credentials: `admin / admin` (change after first login)

---

##  Prometheus Scrape Jobs

| Job | Protocol | Targets |
|-----|----------|---------|
| `prometheus` | HTTP | Self-scrape |
| `node_exporter` | HTTP | Local container, 2× local VMs, EC2 instance |
| `nginx` | HTTP | nginx-exporter container |
| `apache` | HTTP | apache-exporter container |
| `cadvisor` | HTTP | cadvisor container |
| `blackbox_exporter` | HTTP | Blackbox self-scrape |
| `icmp_ping` | ICMP | Gateway, 8.8.8.8, local hosts |
| `blackbox-http` | HTTP | Apache virtual hosts (site1, site2, site3) |
| `blackbox-https` | HTTPS | External URLs (google.com, custom domain) |
| `ssh_probe` | SSH | Remote servers on port 22 |
| `freeipa_tcp` | TCP | FreeIPA ports: 88, 389, 443, 464, 53 |
| `freeipa_ldaps` | LDAPS | FreeIPA LDAP over SSL on port 636 |

---

## 📁 Project Structure

```
monitor-containers/
├── docker-compose.yml          # Full stack definition
├── alertmanager.yml            # Alert routing & email config 
├── nginx.conf                  # NGINX configuration
├── prometheus/
│   ├── prometheus.yml          # Scrape targets & config
│   └── rules.yml               # Alerting rules
├── blackbox/
│   └── blackbox.yml            # Blackbox probe module definitions
├── apache/
│   ├── httpd.conf              # Apache virtual host configuration
│   └── htdocs/                 # Static web content for virtual hosts
├── grafana/
│   ├── provisioning/
│   │   └── datasources/        # Auto-provisioned data sources
│   └── dashboards/             # 9 pre-built Grafana dashboards
├── ansible/
│   ├── inventory/              # Target hosts
│   ├── group_vars/             # Group variables
│   ├── main.yml                # Main playbook
│   └── roles/
│       └── node_exporter/      # Node Exporter deployment role

```

---

## 📜 License

MIT — see [LICENSE](LICENSE)
