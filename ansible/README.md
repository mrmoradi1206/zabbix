# Ansible — Zabbix Agent 2 (Ubuntu 24.04)

Automates installation and configuration of **Zabbix Agent 2** on Debian/Ubuntu (tested on Ubuntu 24.04 “Noble”).  
It adds the official Zabbix repository, installs `zabbix-agent2`, deploys a minimal config, and enables the service.

---

## 🚀 Features
- ✅ Installs official Zabbix apt repo (7.4)
- ✅ Installs `zabbix-agent2`
- ✅ Templates `zabbix_agent2.conf` with your server/active server/hostname
- ✅ Enables & starts the service
- ✅ (Optional) Opens UFW port `10050/tcp`

---

## 🧩 Requirements
- **Ansible 2.12+**
- **Debian/Ubuntu** target hosts (tested on Ubuntu 24.04)
- **SSH access** with sudo privileges (`become: true`)

---

## 🧱 Inventory Example

```ini
[zabbix_agents]
agent-01 ansible_host=10.10.10.11
agent-02 ansible_host=10.10.10.12
```

---

## ⚙️ Variables

| Variable | Default | Description |
|-----------|----------|-------------|
| `zabbix_repo_deb_url` | 7.4 Ubuntu 24.04 .deb URL | Zabbix repo package to install |
| `zabbix_repo_deb_path` | `/opt/zabbix-release_latest_7.4+ubuntu24.04_all.deb` | Where the .deb is downloaded |
| `zabbix_server` | `192.0.2.10` | Zabbix **Server** (passive checks) |
| `zabbix_server_active` | `192.0.2.10` | Zabbix **Active** server |
| `zabbix_agent_hostname` | `{{ inventory_hostname }}` | Agent hostname reported to Zabbix |
| `open_ufw_port` | `true` | Open UFW `10050/tcp` if UFW is active |

Override them in your playbook or `group_vars/zabbix_agents.yml`:

```yaml
zabbix_server: "10.10.10.50"
zabbix_server_active: "10.10.10.50"
zabbix_agent_hostname: "{{ ansible_hostname }}"
open_ufw_port: true
```

---

## ▶️ Usage

```bash
# 1) Clone this repo
git clone https://github.com/<you>/<repo>.git
cd <repo>

# 2) Update inventory and variables
vim inventory.ini

# 3) Run the playbook
ansible-playbook -i inventory.ini zabbix-agent2.yml
```

Or to limit to a single host:

```bash
ansible-playbook -i inventory.ini zabbix-agent2.yml -l agent-01 -K
```

---

## 📝 Template Example

**`templates/zabbix_agent2.conf.j2`**

```jinja
### Managed by Ansible ###
PidFile=/var/run/zabbix/zabbix_agent2.pid
LogType=file
LogFile=/var/log/zabbix/zabbix_agent2.log
LogFileSize=0
Server={{ zabbix_server }}
ServerActive={{ zabbix_server_active }}
Hostname={{ zabbix_agent_hostname }}
Include=/etc/zabbix/zabbix_agent2.d/*.conf
ControlSocket=/tmp/agent.sock
```

---

## ✅ Verification

After playbook runs successfully:

```bash
systemctl status zabbix-agent2
ss -tnlp | grep 10050        # or netstat -tlnp
sudo tail -f /var/log/zabbix/zabbix_agent2.log
```

Then go to **Zabbix Web UI → Configuration → Hosts**,  
and add a host with:
- Interface: Agent, port `10050`
- Hostname same as `Hostname=` in config

---

## 🧰 Troubleshooting

| Issue | Fix |
|-------|-----|
| Repo install fails | Ensure `.deb` URL matches your OS release |
| Connection refused | Check firewalls (port 10050/tcp) |
| Host not showing | Hostname mismatch between agent and UI |
| UFW inactive | Skipped automatically if not enabled |
| Other OS | Change repo URL to match your version (e.g. 22.04) |

---

## 🛠️ Roadmap
- [ ] Autoregistration with active checks  
- [ ] TLS/PSK between agent and server  
- [ ] Role-ify into `roles/zabbix_agent2/`
