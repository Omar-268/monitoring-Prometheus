# 🖥️ Container Monitoring Stack

A production-style container monitoring stack built with **Prometheus**, **Grafana**, **Alertmanager**, **Node Exporter**, **NGINX**, and **cAdvisor** — all orchestrated with Docker Compose.

> Extended from a base monitoring project with NGINX observability, container-level metrics, email alerting, and Wazuh/OpenSearch dashboard integration.

---

## 📐 Architecture

```
                        ┌─────────────────┐
                        │    Grafana :3000 │
                        └────────┬────────┘
                                 │
              ┌──────────────────┼──────────────────┐
              │                  │                  │
   ┌──────────▼──────┐  ┌───────▼────────┐  ┌──────▼──────────┐
   │ Prometheus :9090│  │ Alertmanager   │  │ OpenSearch/Wazuh │
   └──────────┬──────┘  │      :9093     │  │   (external)     │
              │          └───────┬────────┘  └─────────────────┘
    ┌─────────┼──────────┐       │
    │         │          │    Gmail SMTP
┌───▼──┐ ┌───▼────┐ ┌───▼──────────┐
│Node  │ │ NGINX  │ │   cAdvisor   │
│Export│ │Export  │ │    :8080     │
│:9100 │ │ :9113  │ └──────────────┘
└──────┘ └───┬────┘
             │
         ┌───▼────┐
         │ NGINX  │
         │  :80   │
         └────────┘
```

---

## 🧩 Stack Components

| Service | Port | Description |
|---------|------|-------------|
| **Prometheus** | `9090` | Scrapes and stores metrics from all exporters |
| **Grafana** | `3000` | Visualization dashboards |
| **Alertmanager** | `9093` | Handles alerts and sends email notifications |
| **Node Exporter** | `9100` | Host-level metrics (CPU, RAM, disk, network) |
| **NGINX** | `80` | Sample monitored web service |
| **NGINX Exporter** | `9113` | Exposes NGINX metrics for Prometheus |
| **cAdvisor** | `8080` | Per-container resource & performance metrics |

---

## ✨ Key Features

- **Host Monitoring** — CPU, memory, disk, and network via Node Exporter
- **Container Observability** — Per-container CPU/RAM/network via cAdvisor
- **Application Metrics** — Live NGINX traffic (connections, request rates)
- **Email Alerting** — Gmail SMTP alerts via Alertmanager with pre-defined rules
- **Wazuh Integration** — Grafana dashboard connected to OpenSearch/Wazuh for security events
- **Ansible Automation** — Node Exporter deployment automated with Ansible roles

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

Copy the environment template and fill in your credentials:

```bash
cp .env.example .env
```

Edit `.env` with your actual values:

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

> Default Grafana credentials: `admin / admin` (change after first login)

---

## 📁 Project Structure

```
monitor-containers/
├── docker-compose.yml          # Full stack definition
├── alertmanager.yml            # Alert routing & email config (gitignored)
├── nginx.conf                  # NGINX configuration
├── prometheus/
│   ├── prometheus.yml          # Scrape targets & config
│   └── rules.yml               # Alerting rules
├── grafana/
│   ├── provisioning/
│   │   └── datasources/        # Auto-provisioned data sources
│   └── dashboards/             # Pre-built Grafana dashboards
├── ansible/
│   ├── inventory/              # Target hosts
│   ├── site.yml                # Main playbook
│   └── roles/
│       └── node_exporter/      # Node Exporter deployment role
└── .env                        # Secret credentials (gitignored)
```

---

## 🔒 Security Notes

The following files are **gitignored** and must be created manually on each deployment:

| File | Contains |
|------|----------|
| `.env` | OpenSearch password, SMTP credentials |
| `alertmanager.yml` | Gmail SMTP auth password |

Never commit real credentials to version control.

---

## 📜 License

MIT — see [LICENSE](LICENSE)
