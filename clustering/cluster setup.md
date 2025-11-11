# 🧠 Zabbix High Availability (HA) — Two-Node Cluster Setup

This configuration demonstrates a **highly available Zabbix Server cluster** using two `zabbix_server` nodes connected to a **shared database** and optional **shared storage**.

---

## ⚙️ Overview

Each **Zabbix Server node** runs independently but cooperates via the same database backend.  
Zabbix automatically manages the active/standby state between nodes.

| Component | Description |
|------------|-------------|
| **Zabbix Server Nodes** | Two servers: `zabbix-node1`, `zabbix-node2` |
| **Database** | Shared backend (e.g., MySQL Galera Cluster, or PostgreSQL with Patroni/HAProxy) |
| **Optional Shared Storage** | Shared filesystem for `externalscripts/`, `alertscripts/`, and `modules/` |
| **Zabbix Agents** | Configured to send active data to all Zabbix server nodes |

---

## 🏗️ Architecture Diagram

![Zabbix HA Agent](https://github.com/farshadnick/zabbix-course/assets/88557305/67c917f7-be15-4013-bd02-24d907cc529f)

### Logical Flow

```
          ┌──────────────────────────┐
          │ Shared Database Cluster  │  ← MySQL (Galera) / PostgreSQL (Patroni)
          └──────────────────────────┘
                    ▲        ▲
                    │        │
        ┌──────────────────────────────┐
        │      Zabbix Server Nodes      │
        │  zabbix-node1   zabbix-node2 │
        │  (Active)       (Standby)    │
        └──────────────────────────────┘
                    ▲
                    │
          ┌──────────────────────────┐
          │      Zabbix Agents       │
          │  Send to both nodes      │
          └──────────────────────────┘
```

---

## 🧩 Zabbix Server Configuration

Each Zabbix Server node has its own configuration file (`/etc/zabbix/zabbix_server.conf`)  
with a unique **HANodeName** and **NodeAddress**.

### 🖥️ ZBX1 — `/etc/zabbix/zabbix_server.conf`
```ini
HANodeName=zabbix1
NodeAddress=192.168.4.150
```

### 🖥️ ZBX2 — `/etc/zabbix/zabbix_server.conf`
```ini
HANodeName=zabbix2
NodeAddress=192.168.4.151
```

> ⚠️ Make sure both servers use **the same database connection parameters**  
> (DBHost, DBName, DBUser, DBPassword, etc.) and connect to the same database cluster.

---

## 🗄️ Shared Database Backend

The database must be **shared** and **replicated** across both nodes.

You can use:
- **MySQL Galera Cluster**  
  Provides synchronous multi-master replication.
- **PostgreSQL + Patroni + HAProxy**  
  Provides automatic leader election and failover.

Zabbix Server uses the database as the source of truth for cluster state.

---

## 🧠 Optional Shared Storage

If your setup uses any of these:
- `externalscripts/`
- `alertscripts/`
- `modules/`

Then place them on **shared storage** (e.g. NFS, GlusterFS, CephFS)  
and mount the same directory on both servers.

Example mount path:
```
/usr/lib/zabbix/externalscripts/
```

---

## 🔗 Zabbix Agent Configuration

Each **Zabbix Agent** should be configured to send **active checks** to both servers.

### `/etc/zabbix/zabbix_agent2.conf`
```ini
ServerActive=zabbix1.example.com;zabbix2.example.com
```

If you have a standalone local server as well:
```ini
ServerActive=zabbix1.example.com;zabbix2.example.com,localhost
```

This ensures the agent can continue sending data even if one server node is down.

---

## 🔍 Verify HA Cluster Status

You can verify HA node roles and health with:

```bash
zabbix_server -R ha_status
```

Example output:
```
Zabbix HA cluster status:
    Node name: zabbix1
    Node address: 192.168.4.150
    Node status: active
    Node name: zabbix2
    Node address: 192.168.4.151
    Node status: standby
```

---

## ✅ Key Notes

- Only **one node** is **active** at any time — it handles polling, traps, and data collection.  
- Standby nodes automatically take over when the active node fails.  
- The **database cluster** ensures all nodes see consistent configuration and history data.  
- External scripts, modules, and alert handlers should reside on **shared storage** to ensure consistent behavior.

---

## 🧱 Summary

| Layer | Component | Example |
|--------|------------|---------|
| Application | Zabbix Server nodes | `zabbix1`, `zabbix2` |
| Database | Shared DB | MySQL Galera / PostgreSQL Patroni |
| Storage | Optional shared FS | NFS / GlusterFS |
| Agent config | Active checks | `ServerActive=zabbix1;zabbix2` |

---

## 📘 Reference
- [Zabbix Documentation — High Availability](https://www.zabbix.com/documentation/current/en/manual/concepts/high_availability)
- [Zabbix Blog: Zabbix 6.0 HA overview](https://blog.zabbix.com/zabbix-6-0-high-availability/17289/)

---

## 🪪 License
MIT License — see [LICENSE](LICENSE)
