# 🖥️ Container Monitoring Stack

A production-style monitoring stack built with **Prometheus**, **Grafana**, **Alertmanager**, and a suite of exporters — all orchestrated with Docker Compose. Designed for observability across containers, hosts, web servers, and network endpoints.

---

##  Stack Components

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

##  Key Features

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

##  Grafana Dashboards

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

> **⚠️ Note:** The **Wazuh / OpenSearch** dashboard will not display any data unless you have a running Wazuh/OpenSearch instance and have configured its URL and credentials in the Grafana datasource. Without a Wazuh machine, this dashboard will show empty panels or connection errors.

---

##  Getting Started

### Prerequisites
- Docker & Docker Compose
- Git
- Wazuh server (optional)
- FreeIPA server (optional)

### Clone the repository

```bash
git clone https://github.com/Omar-268/monitoring-prometheus.git
cd monitoring-prometheus
```

---

##  Before You Deploy — Full Configuration Checklist

Everything a new user **must** fill in, update, or decide before running `docker compose up -d`.

---

### `.env` — Create & fill credentials

A template is provided  copy it and fill in your values:

```bash
cp .env.example .env
```

---

### `alertmanager.yml` — Fill SMTP credentials

Replace the placeholder values with your real email details:

```yaml
to: 'your_alert_recipient@gmail.com'
from: 'your_sender@gmail.com'
auth_username: 'your_sender@gmail.com'
auth_password: "your_gmail_app_password"
```


---

###  `prometheus/prometheus.yml` — Update all IP addresses & hostnames

The scrape targets are hardcoded to the original lab environment. Replace **every IP and hostname** with your own:

| Section / Job | What to change |
|---------------|----------------|
| `node_exporter` targets | Replace `192.168.106.11`, `192.168.106.12` with your server IPs |
| `icmp_ping` targets | Replace gateway IPs and hosts with IPs you want to ping |
| `blackbox-http` targets | Replace `http://apache/site1/` etc. with your real internal HTTP URLs |
| `ssh_probe` targets | Replace `192.168.106.130:22`, `192.168.106.11:22` with your SSH hosts |
| `freeipa_tcp` targets | Replace `master.ipa.lab` with your FreeIPA server hostname or IP  |
| `freeipa_ldaps` targets | Replace `master.ipa.lab:636` with your LDAPS host  |

> Jobs that point to unreachable hosts will show as `DOWN` in Prometheus but won't break the stack.

---



### Start the stack

```bash
docker compose up -d
```

### Access the dashboards

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

##  Project Structure

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

##  License

MIT — see [LICENSE](LICENSE)
