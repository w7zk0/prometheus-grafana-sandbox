# Prometheus + Grafana Monitoring Stack (Docker)

---

## 📋 Table of Contents

- [Overview](#-overview)
- [Architecture](#-architecture)
- [Skills Demonstrated](#-skills-demonstrated)
- [Prerequisites](#-prerequisites)
- [Quick Start](#-quick-start)
- [Access the UIs](#-access-the-uis)
- [Detailed Configuration](#-detailed-configuration)
- [Recommended Dashboards](#-recommended-dashboards)
- [PromQL Examples](#-promql-examples)
- [Applied Best Practices](#-applied-best-practices)
- [Possible Production Improvements](#-possible-production-improvements)
- [Project Structure](#-project-structure)


---

## 🎯 Overview

This repository contains a **complete monitoring stack** based on:

| Component              | Role                                          | Version  |
|------------------------|-----------------------------------------------|----------|
| **Prometheus**         | Metrics collection, TSDB storage & alerting   | v2.55.1  |
| **Grafana**            | Visualization, dashboards & alerting UI       | 11.3.0   |
| **Node Exporter**      | Host system metrics                           | v1.8.2   |
| **cAdvisor**           | Docker container metrics                      | v0.49.1  |
| **Blackbox Exporter**  | HTTP/HTTPS/TCP/ICMP probing (availability)    | v0.25.0  |
| **Alertmanager**       | Alert routing and management                  | v0.27.0  |

Everything is orchestrated with **Docker Compose**, including persistent volumes, healthchecks, resource limits, and automatic Grafana provisioning.

**Portfolio goal**: demonstrate the ability to set up professional, understandable, maintainable, and securable monitoring.

---

## 🏗 Architecture

<img width="807" height="693" alt="Untitled-2026-09-15-2047" src="https://github.com/user-attachments/assets/19f33168-b549-434d-aef3-f5416772a9f4" />


**Data flow**:
1. Node Exporter and cAdvisor expose metrics in Prometheus format.
2. Prometheus scrapes these targets periodically (every 15s).
3. Grafana queries Prometheus via PromQL to display graphs.
4. Alertmanager receives and routes alerts defined in Prometheus rules.

---

## 💡 Skills Demonstrated

- **Docker & Docker Compose**: multi-service stacks, isolated networks, named volumes, healthchecks, resource limits
- **Prometheus**: scrape configuration, labels, retention, alerting rules, self-monitoring
- **Grafana**: declarative provisioning (datasources + dashboards), basic security
- **Observability**: host + container metrics, symptom-based alerts (CPU, RAM, disk, down)
- **SysAdmin**: understanding of Linux system metrics (CPU, memory, filesystem, network)
- **Best practices**: pinned versions, read-only configs, network isolation, clear documentation

---

## 📦 Prerequisites

- Docker Engine ≥ 24 (or Podman with compose)
- Docker Compose ≥ 2.20 (`docker compose` plugin)
- ~2 GB free RAM
- Free ports: **3000** (Grafana), **9090** (Prometheus), **9093** (Alertmanager), **9100** (Node Exporter), **9115** (Blackbox), **8080** (cAdvisor)

Quick check:

```bash
docker --version
docker compose version
```

---

## 🚀 Quick Start

```bash
# 1. Clone the repository
git clone https://github.com/w7zk0/prometheus-grafana-sandbox.git
cd prometheus-grafana-sandbox

# 2. (Optional) Adjust Grafana password
cp .env.example .env
# Edit .env if needed

# 3. Start the stack
docker compose up -d
# or with Podman:
# podman compose up -d

# 4. Check that everything is healthy
docker compose ps
```

Wait 30–60 seconds for healthchecks to pass.

---

## 🌐 Access the UIs

| Service            | URL                            | Credentials                    |
|--------------------|--------------------------------|--------------------------------|
| **Grafana**        | http://localhost:3000          | `admin` / `admin`              |
| **Prometheus**     | http://localhost:9090          | None (secure in production)    |
| **Alertmanager**   | http://localhost:9093          | -                              |
| **Node Exporter**  | http://localhost:9100/metrics  | -                              |
| **Blackbox**       | http://localhost:9115          | -                              |
| **cAdvisor**       | http://localhost:8080          | -                              |

> ⚠️ **Security**: Change the Grafana password immediately in production and never expose Prometheus / cAdvisor / Alertmanager publicly without authentication or a reverse proxy.

---

## ⚙️ Detailed Configuration

### Prometheus (`prometheus/prometheus.yml`)

- Scrape interval: 15s
- Retention: 15 days / 5 GB max
- Configured jobs:
  - `prometheus` (self-monitoring)
  - `node` (Node Exporter)
  - `cadvisor` (containers)
  - `grafana`
  - `blackbox` + HTTP/TCP probes
  - `alertmanager`

### Alerting rules (`prometheus/rules/alerts.yml`)

Examples included:
- CPU > 80% for 5 min
- Memory > 85%
- Disk space < 15%
- Instance down
- Endpoint down (Blackbox)
- High latency
- SSL certificate expiring soon

Alertmanager is enabled and receives alerts from Prometheus.

### Grafana provisioning

- Prometheus datasource added automatically
- “SysAdmin Portfolio” dashboard folder prepared

### Alertmanager (`alertmanager/alertmanager.yml`)

- Grouping by `alertname`, `severity`, `instance`
- Separate receivers for `critical` and `warning` (Slack/email examples commented out, ready to enable)
- Inhibit rules to reduce noise

---

## 📊 Recommended Dashboards

Once Grafana is running:

1. Go to **Dashboards → Import**
2. Import the following community dashboards (widely used in production):

| ID       | Name                         | Description                          |
|----------|------------------------------|--------------------------------------|
| **1860** | Node Exporter Full           | Most complete host dashboard         |
| **193**  | Docker Monitoring (cAdvisor) | Container view                       |
| **3662** | Prometheus 2.0 Stats         | Prometheus health itself             |
| **7587** | Blackbox Exporter            | Probe success / latency              |

Or create your own panels with the queries below.

---

## 🔍 PromQL Examples

```promql
# Average CPU usage (non-idle)
100 - (avg by(instance) (rate(node_cpu_seconds_total{mode="idle"}[5m])) * 100)

# Available memory in %
(node_memory_MemAvailable_bytes / node_memory_MemTotal_bytes) * 100

# Free disk space in %
(node_filesystem_avail_bytes{fstype!~"tmpfs|overlay"} / node_filesystem_size_bytes) * 100

# Network receive rate
rate(node_network_receive_bytes_total[5m])

# Number of running containers
count(container_last_seen)

# CPU per container
rate(container_cpu_usage_seconds_total{name!=""}[5m]) * 100

# Blackbox probe success
probe_success
```

---

## ✅ Applied Best Practices

| Practice                      | Implementation                                      |
|-------------------------------|-----------------------------------------------------|
| Pinned versions               | Images with specific tags (no `:latest`)            |
| Persistent volumes            | `prometheus_data` + `grafana_data` + `alertmanager_data` |
| Read-only configuration       | `:ro` on config mounts                              |
| Healthchecks                  | All critical services                               |
| Resource limits               | CPU / memory defined                                |
| Isolated network              | Dedicated `monitoring` network                      |
| Declarative provisioning      | Datasource + dashboards via files                   |
| Clear documentation           | README + comments in files                          |
| Basic security                | Grafana password, sign-up disabled                  |

---

## 🔒 Possible Production Improvements

- [x] **Alertmanager** (Slack/email notifications ready to wire)
- [ ] Add a **reverse proxy** (Traefik / Caddy / Nginx) with HTTPS + Basic Auth / OAuth
- [ ] Remote write to Thanos / Cortex / Grafana Cloud for long-term retention
- [x] **Blackbox Exporter** for external HTTP/TCP/ICMP monitoring
- [ ] Instrument applications (MySQL, Redis, Nginx exporters…)
- [ ] Recording rules to pre-aggregate expensive metrics
- [ ] Secrets management (Docker secrets or Vault)
- [ ] Multi-node / Kubernetes deployment (Prometheus Operator)

---

## 📁 Project Structure

```
prometheus-grafana-sandbox/
├── docker-compose.yml              # Stack orchestration
├── .env.example                    # Environment variables
├── prometheus/
│   ├── prometheus.yml              # Main config
│   └── rules/
│       └── alerts.yml              # Alerting rules
├── alertmanager/
│   └── alertmanager.yml            # Alertmanager config
├── blackbox/
│   └── blackbox.yml                # Blackbox modules
├── grafana/
│   └── provisioning/
│       ├── datasources/
│       │   └── datasource.yml      # Auto Prometheus datasource
│       └── dashboards/
│           └── dashboard.yml       # Dashboard provider
├── docs/                           # Extra documentation / GitHub Pages
├── assets/                         # Images / screenshots for the portfolio
└── README.md                       # This file
```

---
