# Ansible Modules

## Ansible Modules — The Building Blocks

A **module** is a small built-in program that performs one specific task.

```
Task:  "Install nginx"
       ↓
Module: yum (handles package installation on RedHat systems)
       ↓
Ansible connects via SSH and runs the module
       ↓
Result: nginx is installed
```

Every task in a playbook uses exactly **one module**.

***

### How to Find Modules

```bash
# Search for a module
ansible-doc -l | grep user

# Read module documentation
ansible-doc yum
ansible-doc copy
ansible-doc user

# See module examples
ansible-doc -e yum
```

Or visit: `docs.ansible.com` → Collections → ansible.builtin

***

## Module 1: `ping`

**Purpose:** Tests if Ansible can connect to a host via SSH.\
It does NOT send an ICMP ping — it just verifies SSH connectivity.

```bash
# Ad-hoc command
ansible all -i inventory/hosts.ini -m ping

# Against specific group
ansible webservers -i inventory/hosts.ini -m ping
```

**Success output:**

```json
web1 | SUCCESS => {
    "changed": false,
    "ping": "pong"
}
```

**Failure output:**

```
web1 | UNREACHABLE! => {
    "msg": "Failed to connect to the host via ssh",
    "unreachable": true
}
```

***

## Module 2: `command` vs `shell`

Both run commands on remote servers. The difference:

| Feature          | `command`       | `shell`          |
| ---------------- | --------------- | ---------------- |
| Pipes `\|`       | ❌ Not supported | ✅ Supported      |
| Redirects `>`    | ❌ Not supported | ✅ Supported      |
| Variables `$VAR` | ❌ Not supported | ✅ Supported      |
| Security         | More secure     | Less secure      |
| Use when         | Simple commands | Complex commands |

### `command` Module Examples

```bash
# Ad-hoc
ansible all -i inventory/hosts.ini -m command -a "uptime"
ansible all -i inventory/hosts.ini -m command -a "hostname"
ansible all -i inventory/hosts.ini -m command -a "df -h"
ansible all -i inventory/hosts.ini -m command -a "free -m"
```

```yaml
# In playbook
- name: Check disk space
  command: df -h

- name: Check uptime
  command: uptime
```

### `shell` Module Examples

```bash
# Ad-hoc — with pipe
ansible all -i inventory/hosts.ini -m shell -a "ps aux | grep nginx"

# Ad-hoc — with variable
ansible all -i inventory/hosts.ini -m shell -a "echo $HOME"
```

```yaml
# In playbook — redirect output to file
- name: Save disk usage to file
  shell: df -h > /tmp/disk_report.txt

- name: Count running processes
  shell: ps aux | wc -l
```

> **Best practice:** Use `command` when possible. Use `shell` only when you need pipes, redirects, or variables.

***

## Module 3: `yum` / `apt` — Package Management

Use `yum` or `dnf` for RedHat/CentOS/Amazon Linux.\
Use `apt` for Ubuntu/Debian.

### `yum` Module Parameters

| Parameter      | Required | Description                   |
| -------------- | -------- | ----------------------------- |
| `name`         | Yes      | Package name                  |
| `state`        | Yes      | `present`, `absent`, `latest` |
| `update_cache` | No       | Update cache before install   |

### State Options Explained

| State     | Meaning                              |
| --------- | ------------------------------------ |
| `present` | Install if not already installed     |
| `absent`  | Remove if installed                  |
| `latest`  | Install OR upgrade to latest version |

```yaml
---
- name: Package Management Practice
  hosts: all
  become: true          # Need root to install packages

  tasks:
    # ─────────────────────────────
    # Install single package
    # ─────────────────────────────
    - name: Install nginx
      yum:
        name: nginx
        state: present

    # ─────────────────────────────
    # Install multiple packages
    # ─────────────────────────────
    - name: Install multiple packages
      yum:
        name:
          - git
          - wget
          - tree
          - vim
          - curl
        state: present

    # ─────────────────────────────
    # Remove a package
    # ─────────────────────────────
    - name: Remove telnet
      yum:
        name: telnet
        state: absent

    # ─────────────────────────────
    # Install latest version
    # ─────────────────────────────
    - name: Ensure vim is latest
      yum:
        name: vim
        state: latest

    # ─────────────────────────────
    # Update ALL packages
    # ─────────────────────────────
    - name: Update all packages
      yum:
        name: "*"
        state: latest
```

### `apt` Module (Ubuntu/Debian)

```yaml
- name: Install nginx on Ubuntu
  apt:
    name: nginx
    state: present
    update_cache: true          # Like running apt update first

- name: Install multiple packages
  apt:
    name:
      - nginx
      - git
      - curl
    state: present
    update_cache: true
```

### Ad-hoc Package Commands

```bash
# Install package
ansible all -i inventory/hosts.ini -m yum -a "name=nginx state=present" -b

# Remove package
ansible all -i inventory/hosts.ini -m yum -a "name=nginx state=absent" -b

# The -b flag means become (use sudo)
```

***

## Module 4: `service` — Manage Services

Controls services on remote servers (start, stop, restart, enable).

### Service Module Parameters

| Parameter | Description    | Values                                        |
| --------- | -------------- | --------------------------------------------- |
| `name`    | Service name   | `nginx`, `httpd`, `sshd`                      |
| `state`   | Desired state  | `started`, `stopped`, `restarted`, `reloaded` |
| `enabled` | Start on boot? | `true`, `false`                               |

### State Values Explained

| State       | What it does                       |
| ----------- | ---------------------------------- |
| `started`   | Start if not running               |
| `stopped`   | Stop if running                    |
| `restarted` | Always restart (stop then start)   |
| `reloaded`  | Reload config without full restart |

```yaml
---
- name: Service Management Practice
  hosts: all
  become: true

  tasks:
    # Start nginx
    - name: Start nginx
      service:
        name: nginx
        state: started

    # Stop nginx
    - name: Stop nginx
      service:
        name: nginx
        state: stopped

    # Restart nginx
    - name: Restart nginx
      service:
        name: nginx
        state: restarted

    # Start AND enable on boot
    - name: Start and enable nginx
      service:
        name: nginx
        state: started
        enabled: true

    # Only enable on boot (don't change running state)
    - name: Enable nginx on boot
      service:
        name: nginx
        enabled: true

    # Disable from boot
    - name: Disable nginx from boot
      service:
        name: nginx
        enabled: false
```

### Ad-hoc Service Commands

```bash
# Start service
ansible all -i inventory/hosts.ini -m service -a "name=nginx state=started" -b

# Check status
ansible all -i inventory/hosts.ini -m command -a "systemctl status nginx" -b
```

***

## Module 5: `file` — Manage Files and Directories

Create, delete, and modify permissions of files and directories.

### File Module Parameters

| Parameter | Description                            | Example            |
| --------- | -------------------------------------- | ------------------ |
| `path`    | File/directory path                    | `/tmp/myfile.txt`  |
| `state`   | `touch`, `directory`, `absent`, `link` | `touch`            |
| `owner`   | File owner                             | `ec2-user`         |
| `group`   | File group                             | `ec2-user`         |
| `mode`    | Permissions                            | `'0644'`, `'0755'` |
| `recurse` | Apply to all contents                  | `true`             |

### Understanding `mode` (Permissions)

```
0644  →  rw-r--r--   (owner can read/write, others read only)
0755  →  rwxr-xr-x   (owner all, others read/execute)
0777  →  rwxrwxrwx   (everyone has all permissions)
0400  →  r--------   (owner read only — for SSH keys)
```

```yaml
---
- name: File Module Practice
  hosts: all
  become: true

  tasks:
    # ─────────────────────────────
    # Create an empty file
    # ─────────────────────────────
    - name: Create empty file
      file:
        path: /tmp/myfile.txt
        state: touch
        owner: ec2-user
        group: ec2-user
        mode: '0644'

    # ─────────────────────────────
    # Create a directory
    # ─────────────────────────────
    - name: Create directory
      file:
        path: /opt/myapp
        state: directory
        mode: '0755'
        owner: ec2-user

    # ─────────────────────────────
    # Create nested directories
    # ─────────────────────────────
    - name: Create nested directories
      file:
        path: /opt/myapp/logs/archive
        state: directory
        recurse: true           # Creates parent dirs too
        mode: '0755'

    # ─────────────────────────────
    # Change permissions of existing file
    # ─────────────────────────────
    - name: Change file permissions
      file:
        path: /tmp/myfile.txt
        mode: '0600'
        owner: root

    # ─────────────────────────────
    # Delete a file
    # ─────────────────────────────
    - name: Delete file
      file:
        path: /tmp/myfile.txt
        state: absent

    # ─────────────────────────────
    # Delete a directory (and contents)
    # ─────────────────────────────
    - name: Delete directory
      file:
        path: /opt/myapp
        state: absent
```

***

## Module 6: `copy` — Copy Files to Remote

Copies files FROM your control node TO remote servers.

### Copy Module Parameters

| Parameter | Description                                    |
| --------- | ---------------------------------------------- |
| `src`     | Source path on control node                    |
| `dest`    | Destination path on remote server              |
| `owner`   | Set file owner on remote                       |
| `group`   | Set file group on remote                       |
| `mode`    | Set permissions on remote                      |
| `backup`  | Backup existing file before overwriting        |
| `content` | Write inline content instead of copying a file |

```yaml
---
- name: Copy Module Practice
  hosts: all
  become: true

  tasks:
    # ─────────────────────────────
    # Basic file copy
    # ─────────────────────────────
    - name: Copy config file
      copy:
        src: /home/ec2-user/nginx.conf     # On control node
        dest: /etc/nginx/nginx.conf        # On remote server
        owner: root
        group: root
        mode: '0644'
        backup: true                       # Keeps old file as .backup

    # ─────────────────────────────
    # Copy with inline content
    # ─────────────────────────────
    - name: Create file with content
      copy:
        content: |
          Welcome to my server
          Managed by Ansible
          Server: {{ inventory_hostname }}
        dest: /tmp/welcome.txt
        mode: '0644'

    # ─────────────────────────────
    # Copy and set permissions
    # ─────────────────────────────
    - name: Copy script file
      copy:
        src: scripts/deploy.sh
        dest: /usr/local/bin/deploy.sh
        owner: root
        group: root
        mode: '0755'                       # Executable
```

### Idempotency in Copy Module

If you run a copy task multiple times:

* File already same → `ok` (no change)
* File is different → `changed` (overwrites)
* `backup: true` → saves old version as `.backup`

***

## Module 7: `user` — Manage Users

Create, modify, and delete users on remote servers.

```yaml
---
- name: User Management
  hosts: all
  become: true

  tasks:
    # ─────────────────────────────
    # Create a basic user
    # ─────────────────────────────
    - name: Create user devops
      user:
        name: devops
        comment: "DevOps Engineer"
        shell: /bin/bash
        home: /home/devops
        create_home: true

    # ─────────────────────────────
    # Create user and add to groups
    # ─────────────────────────────
    - name: Create user and add to groups
      user:
        name: webadmin
        groups:
          - wheel          # sudo access
          - nginx
        append: true       # Don't remove from other groups

    # ─────────────────────────────
    # Create user with password
    # ─────────────────────────────
    - name: Create user with password
      user:
        name: appuser
        password: "{{ 'SecurePass123!' | password_hash('sha512') }}"
        update_password: always   # Update even if already set

    # ─────────────────────────────
    # Delete user
    # ─────────────────────────────
    - name: Remove user
      user:
        name: devops
        state: absent
        remove: true        # Also deletes home directory
```

### Verify User Was Created

```bash
# Check on remote server
ansible all -i inventory/hosts.ini -m command -a "id devops"
ansible all -i inventory/hosts.ini -m command -a "cat /etc/passwd | grep devops"
```

***

## Module 8: `cron` — Schedule Jobs

Manage cron jobs on remote servers.

### Cron Time Format

```
┌─────── minute (0-59)
│ ┌───── hour (0-23)
│ │ ┌─── day of month (1-31)
│ │ │ ┌─ month (1-12)
│ │ │ │ ┌ day of week (0-7, 0=Sunday)
│ │ │ │ │
* * * * *  command

Examples:
0 2 * * *     → 2:00 AM every day
*/5 * * * *   → Every 5 minutes
0 0 * * 0     → Every Sunday at midnight
30 18 * * 1-5 → 6:30 PM on weekdays
```

```yaml
---
- name: Cron Job Management
  hosts: all
  become: true

  tasks:
    # ─────────────────────────────
    # Add a cron job
    # ─────────────────────────────
    - name: Schedule daily backup at 2 AM
      cron:
        name: "daily backup"         # Unique identifier (required)
        minute: "0"
        hour: "2"
        day: "*"
        month: "*"
        weekday: "*"
        user: ec2-user
        job: "/usr/local/bin/backup.sh >> /var/log/backup.log 2>&1"

    # ─────────────────────────────
    # Every 5 minutes
    # ─────────────────────────────
    - name: Health check every 5 minutes
      cron:
        name: "health check"
        minute: "*/5"
        user: ec2-user
        job: "/usr/local/bin/health.sh"

    # ─────────────────────────────
    # Disable a cron job (comment it out)
    # ─────────────────────────────
    - name: Disable health check
      cron:
        name: "health check"
        disabled: true

    # ─────────────────────────────
    # Remove a cron job
    # ─────────────────────────────
    - name: Remove backup job
      cron:
        name: "daily backup"
        state: absent
```

### Verify Cron Jobs

```bash
# Check cron jobs for a user on remote server
ansible all -i inventory/hosts.ini -m command -a "crontab -l" --become-user=ec2-user
```

***

## Module 9: `get_url` — Download Files

Download files from the internet directly onto remote servers.

```yaml
---
- name: Download Files
  hosts: all
  become: true

  tasks:
    # ─────────────────────────────
    # Basic download
    # ─────────────────────────────
    - name: Download Python installer
      get_url:
        url: "https://www.python.org/ftp/python/3.11.0/Python-3.11.0.tgz"
        dest: /tmp/python.tgz
        mode: '0644'

    # ─────────────────────────────
    # Download with owner and permissions
    # ─────────────────────────────
    - name: Download app binary
      get_url:
        url: "https://example.com/myapp-v1.0.tar.gz"
        dest: /opt/myapp.tar.gz
        owner: ec2-user
        group: ec2-user
        mode: '0755'

    # ─────────────────────────────
    # Download with checksum validation
    # ─────────────────────────────
    - name: Download verified file
      get_url:
        url: "https://example.com/important.jar"
        dest: /opt/important.jar
        checksum: "sha256:a1b2c3d4..."     # Validates file integrity
```

***

## Module 10: `firewalld` — Firewall Management

Manage firewall rules on RedHat/CentOS systems.

```yaml
---
- name: Firewall Management
  hosts: all
  become: true

  tasks:
    # ─────────────────────────────
    # Allow HTTP service
    # ─────────────────────────────
    - name: Allow HTTP traffic
      firewalld:
        service: http
        state: enabled
        permanent: true          # Survives reboot

    # ─────────────────────────────
    # Allow HTTPS service
    # ─────────────────────────────
    - name: Allow HTTPS traffic
      firewalld:
        service: https
        state: enabled
        permanent: true

    # ─────────────────────────────
    # Open custom port
    # ─────────────────────────────
    - name: Open port 8080
      firewalld:
        port: 8080/tcp
        state: enabled
        permanent: true

    # ─────────────────────────────
    # Block a port
    # ─────────────────────────────
    - name: Block port 23 (telnet)
      firewalld:
        port: 23/tcp
        state: disabled
        permanent: true

    # ─────────────────────────────
    # Reload firewall to apply changes
    # ─────────────────────────────
    - name: Reload firewall
      service:
        name: firewalld
        state: reloaded
```

***

## Module 11: `setup` — Gather System Facts

This module **auto-runs** at the start of every playbook.\
It collects detailed info about the remote system.

```bash
# See ALL facts
ansible all -i inventory/hosts.ini -m setup

# Filter specific facts
ansible all -i inventory/hosts.ini -m setup -a "filter=ansible_os_family"
ansible all -i inventory/hosts.ini -m setup -a "filter=ansible_distribution*"
ansible all -i inventory/hosts.ini -m setup -a "filter=ansible_memory_mb"
ansible all -i inventory/hosts.ini -m setup -a "filter=ansible_processor*"
```

### Most Useful Facts

```
ansible_os_family                 → RedHat / Debian
ansible_distribution              → CentOS / Ubuntu / Amazon
ansible_distribution_version      → 7.9 / 20.04
ansible_distribution_major_version→ 7 / 20
ansible_hostname                  → server1
ansible_fqdn                      → server1.example.com
ansible_default_ipv4.address      → 192.168.1.10
ansible_memory_mb.real.total      → 8192 (MB)
ansible_processor_vcpus           → 4
ansible_architecture              → x86_64
ansible_kernel                    → 5.10.68
```

### Use Facts in a Playbook

```yaml
---
- name: Show server facts
  hosts: all

  tasks:
    - name: Print OS info
      debug:
        msg: "OS is {{ ansible_distribution }} {{ ansible_distribution_version }}"

    - name: Print memory
      debug:
        msg: "Total RAM: {{ ansible_memory_mb.real.total }} MB"

    - name: Print IP address
      debug:
        msg: "IP Address: {{ ansible_default_ipv4.address }}"
```

***

## 2.15 Ad-Hoc Commands Reference

Ad-hoc commands are **one-off tasks** without writing a playbook.

### Syntax

```bash
ansible <host-pattern> -i <inventory> -m <module> -a "<arguments>" [flags]
```

### Common Flags

| Flag                | Meaning                  |
| ------------------- | ------------------------ |
| `-m`                | Module name              |
| `-a`                | Module arguments         |
| `-i`                | Inventory file           |
| `-b`                | Become (use sudo)        |
| `--ask-become-pass` | Prompt for sudo password |
| `-v`                | Verbose output           |
| `-vv`               | More verbose             |
| `-vvv`              | Very verbose (debug)     |

### Quick Reference

```bash
# ─── CONNECTIVITY ───────────────────────────────
ansible all -i inventory/hosts.ini -m ping

# ─── SYSTEM INFO ────────────────────────────────
ansible all -i inventory/hosts.ini -m command -a "uptime"
ansible all -i inventory/hosts.ini -m command -a "hostname"
ansible all -i inventory/hosts.ini -m command -a "df -h"
ansible all -i inventory/hosts.ini -m command -a "free -m"
ansible all -i inventory/hosts.ini -m shell   -a "cat /etc/os-release"

# ─── PACKAGES ───────────────────────────────────
ansible all -i inventory/hosts.ini -m yum -a "name=vim state=present" -b
ansible all -i inventory/hosts.ini -m yum -a "name=vim state=absent" -b

# ─── SERVICES ───────────────────────────────────
ansible all -i inventory/hosts.ini -m service -a "name=nginx state=started" -b
ansible all -i inventory/hosts.ini -m service -a "name=nginx state=stopped" -b
ansible all -i inventory/hosts.ini -m service -a "name=nginx state=restarted" -b

# ─── FILES ──────────────────────────────────────
ansible all -i inventory/hosts.ini -m file -a "path=/tmp/test.txt state=touch"
ansible all -i inventory/hosts.ini -m file -a "path=/tmp/mydir state=directory"
ansible all -i inventory/hosts.ini -m file -a "path=/tmp/test.txt state=absent"

# ─── COPY ───────────────────────────────────────
ansible all -i inventory/hosts.ini -m copy \
  -a "src=/tmp/local.txt dest=/tmp/remote.txt mode=0644"

# ─── USERS ──────────────────────────────────────
ansible all -i inventory/hosts.ini -m user -a "name=testuser state=present" -b
ansible all -i inventory/hosts.ini -m user -a "name=testuser state=absent" -b

# ─── WITH SUDO ──────────────────────────────────
ansible all -i inventory/hosts.ini -m yum -a "name=git state=present" \
  -b --ask-become-pass
```

***
