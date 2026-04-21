# 3 Core Components of Ansible

## The 3 Core Components

```
┌─────────────────────────────────────────────┐
│              ANSIBLE WORKS WITH              │
│                                             │
│  INVENTORY     PLAYBOOK      MODULES        │
│  ─────────     ────────      ───────        │
│  "WHERE"       "WHAT"        "HOW"          │
│  List of       YAML file     Built-in       │
│  servers       with tasks    programs       │
│                                             │
└─────────────────────────────────────────────┘
```

### Component 1: Inventory

A file listing all your servers (IP addresses or hostnames).

```ini
# Simple inventory
192.168.1.10
192.168.1.11
192.168.1.12
```

### Component 2: Playbook

A YAML file describing what tasks to perform on which servers.

```yaml
---
- name: Install Nginx on web servers
  hosts: webservers
  tasks:
    - name: Install nginx
      yum:
        name: nginx
        state: present
```

### Component 3: Modules

Small built-in programs that do specific tasks.

```
yum      → install/remove packages (RedHat/CentOS)
apt      → install/remove packages (Ubuntu/Debian)
service  → start/stop services
copy     → copy files to remote servers
user     → manage users
file     → create/delete files and directories
```
