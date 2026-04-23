# Multi-Play Playbook

## 3.16 Multi-Play Playbook

One playbook file can contain multiple plays targeting different hosts.

```yaml
---
# ═══════════════════════════════
# PLAY 1: Setup Web Servers
# ═══════════════════════════════
- name: Setup Web Servers
  hosts: webservers
  become: true

  tasks:
    - name: Install nginx
      yum:
        name: nginx
        state: present

    - name: Start nginx
      service:
        name: nginx
        state: started
        enabled: true

# ═══════════════════════════════
# PLAY 2: Setup Database Servers
# ═══════════════════════════════
- name: Setup Database Servers
  hosts: dbservers
  become: true

  tasks:
    - name: Install MySQL
      yum:
        name: mysql-server
        state: present

    - name: Start MySQL
      service:
        name: mysqld
        state: started
        enabled: true

# ═══════════════════════════════
# PLAY 3: Common setup for ALL
# ═══════════════════════════════
- name: Common Server Setup
  hosts: all
  become: true

  tasks:
    - name: Install common tools
      yum:
        name:
          - vim
          - git
          - curl
          - wget
        state: present

    - name: Set timezone
      timezone:
        name: Asia/Kolkata
```

***

