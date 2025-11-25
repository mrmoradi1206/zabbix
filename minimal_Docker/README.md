# Zabbix 7 + PostgreSQL 16 (Docker Compose Deployment)

A clean, production-ready Docker Compose stack for running **Zabbix 7**
with **PostgreSQL 16** and the official **Zabbix Web (Nginx + PHP-FPM)**
interface.

This repository is designed to be simple, reliable, and ready for small
VPS deployments, home labs, or enterprise testing.

------------------------------------------------------------------------

## 🚀 Features

-   ✔️ Zabbix Server 7.0 LTS\
-   ✔️ Zabbix Web Frontend (Nginx + PHP-FPM)\
-   ✔️ PostgreSQL 16 (official recommended DB)\
-   ✔️ Includes healthchecks, tuning, and persistent volumes\
-   ✔️ Low-memory parameters for small VPS servers\
-   ✔️ Full upgrade instructions included\
-   ✔️ Zero secrets in repository (safe for GitHub)

------------------------------------------------------------------------

## 📦 Contents

    docker-compose.yml
    README.md
    data/
     ├── db/                # PostgreSQL data
     ├── alertscripts/      # Custom alert scripts
     ├── externalscripts/   # Custom Zabbix external scripts
     └── web/               # SSL certs (optional)

> All data lives under `./data` --- safe to back up and restore.

------------------------------------------------------------------------

## 📥 Installation

### 1. Clone the repository

``` bash
git clone https://github.com/<your-user>/<your-repo>.git
cd <your-repo>
```

### 2. Start the stack

``` bash
docker compose up -d
```

### 3. Access Zabbix Web

Open:

    http://<your-server-ip>:8080

Default Zabbix login:

    User: Admin
    Password: zabbix

You will be asked to change the password on first login.

------------------------------------------------------------------------

## 🧩 Services Overview

### **1. PostgreSQL 16**

-   Stores all Zabbix configuration, events, history, trends.
-   Optimized parameters for small VPS (shared_buffers, cache sizes,
    etc.).

### **2. Zabbix Server 7**

-   Collects monitoring data.
-   Talks to agents, proxies, and SNMP devices.
-   Cached and tuned for memory-limited environments.

### **3. Zabbix Web (Nginx + PHP-FPM)**

-   Web UI to view metrics, dashboards, hosts, triggers, etc.
-   Runs on port `8080`.

------------------------------------------------------------------------

## 🔧 Customization

### Change PHP Timezone

Edit inside `docker-compose.yml`:

``` yaml
PHP_TZ: "Europe/Amsterdam"
```

Examples: - `Asia/Tehran` - `Europe/Berlin` - `UTC`

------------------------------------------------------------------------

## 📂 Persistent Data

Everything you need is stored in `./data`:

  Folder                   What It Contains
  ------------------------ ------------------------------------
  `data/db`                PostgreSQL database (critical)
  `data/alertscripts`      Zabbix alert scripts
  `data/externalscripts`   Custom scripts executed by Zabbix
  `data/web`               SSL certificates if you want HTTPS

Back up `data/db` regularly.

------------------------------------------------------------------------

## 🛟 Backup Instructions

### Database backup (recommended)

``` bash
docker exec -t zabbix-postgres  pg_dump -U zabbix -d zabbix  | gzip > zabbix_pg_backup_$(date +%F).sql.gz
```

### Full backup (config + DB)

``` bash
tar czf zabbix_stack_backup_$(date +%F).tar.gz .
```

------------------------------------------------------------------------

## 🔄 Upgrading Zabbix to Newer 7.x Versions

1.  Stop the stack:

``` bash
docker compose down
```

2.  Edit `docker-compose.yml` and change image tags:

``` yaml
zabbix/zabbix-server-pgsql:alpine-7.0-latest
zabbix/zabbix-web-nginx-pgsql:alpine-7.0-latest
```

For example:

-   `alpine-7.0-latest` → `alpine-7.2-latest`

3.  Pull new images:

``` bash
docker compose pull
```

4.  Start:

``` bash
docker compose up -d
```

5.  Watch Zabbix server logs to follow DB schema upgrade:

``` bash
docker logs -f zabbix-server
```

------------------------------------------------------------------------

## 🧹 Removing the Stack

Stop without deleting data:

``` bash
docker compose down
```

Stop **and delete all data** (DANGER):

``` bash
docker compose down -v
```

------------------------------------------------------------------------

## ❓ Need Help?

This deployment is designed for:

-   Home labs\
-   Monitoring Docker hosts\
-   Monitoring SNMP switches/routers\
-   VPS with low RAM (1--2 GB)\
-   Zabbix training environments\
-   Small business production use

If you need: - Grafana integration\
- Prometheus integration\
- HA Zabbix setup\
- Auto backup scripts\
- Docker monitoring template

Just ask --- happy to help.

------------------------------------------------------------------------

## 🔚 Summary

This repository gives you a clean, safe, and modern Zabbix 7 deployment
with:

-   PostgreSQL 16\
-   Persistent data\
-   Low-memory tuning\
-   No hardcoded secrets\
-   Easy upgrades\
-   GitHub-safe structure

Deploy it anywhere in seconds using Docker Compose.

    docker compose up -d

Enjoy monitoring! 🎉
