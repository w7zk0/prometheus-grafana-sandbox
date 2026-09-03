# 🚀 Portfolio SysAdmin – Stack Monitoring Prometheus + Grafana (Docker)

> **Projet de démonstration** des compétences en monitoring, observability et administration système.  
> Stack complète, prête à déployer, suivant les bonnes pratiques Docker & Prometheus.

[![Docker](https://img.shields.io/badge/Docker-Compose-2496ED?logo=docker&logoColor=white)](https://www.docker.com/)
[![Prometheus](https://img.shields.io/badge/Prometheus-E6522C?logo=prometheus&logoColor=white)](https://prometheus.io/)
[![Grafana](https://img.shields.io/badge/Grafana-F46800?logo=grafana&logoColor=white)](https://grafana.com/)
[![License](https://img.shields.io/badge/License-MIT-green.svg)](LICENSE)

---

## 📋 Table des matières

- [Présentation](#-présentation)
- [Architecture](#-architecture)
- [Compétences démontrées](#-compétences-démontrées)
- [Prérequis](#-prérequis)
- [Démarrage rapide](#-démarrage-rapide)
- [Accès aux interfaces](#-accès-aux-interfaces)
- [Configuration détaillée](#-configuration-détaillée)
- [Dashboards recommandés](#-dashboards-recommandés)
- [Exemples de requêtes PromQL](#-exemples-de-requêtes-promql)
- [Bonnes pratiques appliquées](#-bonnes-pratiques-appliquées)
- [Améliorations possibles (production)](#-améliorations-possibles-production)
- [Structure du projet](#-structure-du-projet)
- [Auteur](#-auteur)

---

## 🎯 Présentation

Ce repository contient une **stack de monitoring complète** basée sur :

| Composant          | Rôle                                      | Version    |
|--------------------|-------------------------------------------|------------|
| **Prometheus**     | Collecte, stockage TSDB & alerting        | v2.55.1    |
| **Grafana**        | Visualisation, dashboards & alerting UI   | 11.3.0     |
| **Node Exporter**  | Métriques système de l’hôte               | v1.8.2     |
| **cAdvisor**       | Métriques des conteneurs Docker           | v0.49.1    |

Le tout est orchestré avec **Docker Compose**, avec volumes persistants, healthchecks, limites de ressources et provisioning automatique de Grafana.

**Objectif portfolio** : montrer que je sais mettre en place un monitoring professionnel, compréhensible, maintenable et sécurisable.

---

## 🏗 Architecture

```
┌─────────────────────────────────────────────────────────────┐
│                     Hôte Linux / Docker                      │
│                                                             │
│  ┌──────────────┐     scrape      ┌──────────────────────┐  │
│  │ Node Exporter│◄────────────────┤                      │  │
│  │   :9100      │                 │                      │  │
│  └──────────────┘                 │     Prometheus       │  │
│                                   │       :9090          │  │
│  ┌──────────────┐     scrape      │                      │  │
│  │   cAdvisor   │◄────────────────┤  - scrape configs    │  │
│  │   :8080      │                 │  - rules / alerts    │  │
│  └──────────────┘                 │  - TSDB (15j / 5Go)  │  │
│                                   └──────────┬───────────┘  │
│                                              │              │
│                                              │ query        │
│                                              ▼              │
│                                   ┌──────────────────────┐  │
│                                   │      Grafana         │  │
│                                   │       :3000          │  │
│                                   │  - Datasource auto   │  │
│                                   │  - Dashboards        │  │
│                                   └──────────────────────┘  │
└─────────────────────────────────────────────────────────────┘
```

**Flux de données** :
1. Node Exporter et cAdvisor exposent les métriques en format Prometheus.
2. Prometheus scrape périodiquement (15s) ces targets.
3. Grafana interroge Prometheus via PromQL pour afficher les graphiques.

---

## 💡 Compétences démontrées

- **Docker & Docker Compose** : multi-services, networks isolés, volumes nommés, healthchecks, resource limits
- **Prometheus** : configuration de scrape, labels, retention, rules d’alerting, self-monitoring
- **Grafana** : provisioning déclaratif (datasources + dashboards), sécurité basique
- **Observability** : métriques host + conteneurs, alertes sur symptômes (CPU, RAM, disque, down)
- **SysAdmin** : compréhension des métriques système Linux (CPU, mémoire, filesystem, réseau)
- **Bonnes pratiques** : versions pinnées, configuration en lecture seule, isolation réseau, documentation claire

---

## 📦 Prérequis

- Docker Engine ≥ 24
- Docker Compose ≥ 2.20 (`docker compose` plugin)
- ~2 Go de RAM libre
- Ports libres : **3000** (Grafana), **9090** (Prometheus), **9100** (Node Exporter), **8080** (cAdvisor)

Vérification rapide :

```bash
docker --version
docker compose version
```

---

## 🚀 Démarrage rapide

```bash
# 1. Cloner le dépôt
git clone https://github.com/w7zk0/prometheus-grafana-portfolio.git
cd prometheus-grafana-portfolio

# 2. (Optionnel) Adapter le mot de passe Grafana
cp .env.example .env
# Éditer .env si besoin

# 3. Lancer la stack
docker compose up -d

# 4. Vérifier que tout est healthy
docker compose ps
```

Attendre 30-60 secondes que les healthchecks passent.

---

## 🌐 Accès aux interfaces

| Service          | URL                      | Identifiants                  |
|------------------|--------------------------|-------------------------------|
| **Grafana**      | http://localhost:3000    | `admin` / `PortfolioAdmin2024!` |
| **Prometheus**   | http://localhost:9090    | Aucun (à sécuriser en prod)   |
| **Node Exporter**| http://localhost:9100/metrics | -                          |
| **cAdvisor**     | http://localhost:8080    | -                             |

> ⚠️ **Sécurité** : changez immédiatement le mot de passe Grafana en production et ne exposez jamais Prometheus/cAdvisor publiquement sans authentification / reverse proxy.

---

## ⚙️ Configuration détaillée

### Prometheus (`prometheus/prometheus.yml`)

- Scrape interval : 15s
- Retention : 15 jours / 5 Go max
- Jobs configurés :
  - `prometheus` (self-monitoring)
  - `node` (Node Exporter)
  - `cadvisor` (conteneurs)
  - `grafana`

### Règles d’alerting (`prometheus/rules/alerts.yml`)

Exemples inclus :
- CPU > 80 % pendant 5 min
- Mémoire > 85 %
- Espace disque < 15 %
- Instance down

Pour activer Alertmanager, décommentez la section `alerting` et ajoutez le service dans le `docker-compose.yml`.

### Grafana provisioning

- Datasource Prometheus ajoutée automatiquement
- Dossier de dashboards « SysAdmin Portfolio » préparé

---

## 📊 Dashboards recommandés

Une fois Grafana lancé :

1. Allez dans **Dashboards → Import**
2. Importez les dashboards communautaires suivants (très utilisés en production) :

| ID     | Nom                          | Description                          |
|--------|------------------------------|--------------------------------------|
| **1860** | Node Exporter Full          | Dashboard le plus complet pour host  |
| **193**  | Docker Monitoring (cAdvisor)| Vue des conteneurs                   |
| **3662** | Prometheus 2.0 Stats        | Santé de Prometheus lui-même         |

Ou créez vos propres panels avec les requêtes ci-dessous.

---

## 🔍 Exemples de requêtes PromQL

```promql
# Utilisation CPU moyenne (non-idle)
100 - (avg by(instance) (rate(node_cpu_seconds_total{mode="idle"}[5m])) * 100)

# Mémoire disponible en %
(node_memory_MemAvailable_bytes / node_memory_MemTotal_bytes) * 100

# Espace disque libre en %
(node_filesystem_avail_bytes{fstype!~"tmpfs|overlay"} / node_filesystem_size_bytes) * 100

# Charge réseau (réception)
rate(node_network_receive_bytes_total[5m])

# Nombre de conteneurs en cours
count(container_last_seen)

# CPU par conteneur
rate(container_cpu_usage_seconds_total{name!=""}[5m]) * 100
```

---

## ✅ Bonnes pratiques appliquées

| Pratique                        | Implémentation                                      |
|---------------------------------|-----------------------------------------------------|
| Versions pinnées                | Images avec tags précis (pas de `:latest`)          |
| Volumes persistants             | `prometheus_data` + `grafana_data`                  |
| Configuration en lecture seule  | `:ro` sur les montages de config                    |
| Healthchecks                    | Tous les services critiques                         |
| Limites de ressources           | CPU / mémoire définis                               |
| Réseau isolé                    | Network `monitoring` dédié                          |
| Provisioning déclaratif         | Datasource + dashboards via fichiers                |
| Documentation claire            | README + commentaires dans les fichiers             |
| Sécurité de base                | Mot de passe Grafana, pas d’inscription ouverte     |

---

## 🔒 Améliorations possibles (niveau production)

- [ ] Ajouter **Alertmanager** + notifications (Slack, email, PagerDuty)
- [ ] Mettre un **reverse proxy** (Traefik / Caddy / Nginx) avec HTTPS + Basic Auth / OAuth
- [ ] Remote write vers Thanos / Cortex / Grafana Cloud pour rétention longue
- [ ] Ajouter **Blackbox Exporter** pour le monitoring HTTP/TCP/ICMP externe
- [ ] Instrumenter des applications (exporters MySQL, Redis, Nginx…)
- [ ] Recording rules pour pré-agréger les métriques coûteuses
- [ ] Secrets management (Docker secrets ou Vault)
- [ ] Déploiement multi-nœuds / Kubernetes (Prometheus Operator)

---

## 📁 Structure du projet

```
prometheus-grafana-portfolio/
├── docker-compose.yml              # Orchestration de la stack
├── .env.example                    # Variables d'environnement
├── prometheus/
│   ├── prometheus.yml              # Config principale
│   └── rules/
│       └── alerts.yml              # Règles d'alerting
├── grafana/
│   └── provisioning/
│       ├── datasources/
│       │   └── datasource.yml      # Auto-config Prometheus
│       └── dashboards/
│           └── dashboard.yml       # Provider de dashboards
├── docs/                           # Documentation complémentaire
├── assets/                         # Images / captures pour le portfolio
└── README.md                       # Ce fichier
```

---

## 👤 Auteur

**w7zk0**  
SysAdmin / DevOps / SRE en devenir  

- GitHub : https://github.com/w7zk0
- LinkedIn : [à compléter]
- Email : [à compléter]

> Ce projet fait partie de mon portfolio freelance IT.  
> N’hésitez pas à me contacter pour des missions de monitoring, infrastructure as code ou optimisation système.

---

## 📄 Licence

MIT – Libre d’utilisation et de modification (avec attribution appréciée).

---

**Merci d’avoir consulté ce projet !**  
Si vous le trouvez utile, un ⭐ sur GitHub fait toujours plaisir 😊
