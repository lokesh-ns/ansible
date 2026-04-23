# Ansible Inventory File

## Inventory File — Deep Dive

### What is an Inventory File?

An inventory file is a **list of servers** that Ansible manages.\
Think of it as your address book of servers.

Without an inventory, Ansible doesn't know which servers to connect to.

### Default Location

```
/etc/ansible/hosts
```

You can also create your own inventory file anywhere and pass it with `-i`:

```bash
ansible-playbook -i /path/to/my/inventory playbook.yml
```

***

### Inventory File Formats

Ansible supports two formats:

#### Format 1: INI (Most Common for Beginners)

```ini
# ─────────────────────────────────────
# UNGROUPED HOSTS (no group assigned)
# ─────────────────────────────────────
192.168.1.5
web.example.com

# ─────────────────────────────────────
# GROUPED HOSTS
# ─────────────────────────────────────
[webservers]
web1 ansible_host=192.168.1.10
web2 ansible_host=192.168.1.11

[dbservers]
db1 ansible_host=192.168.1.20
db2 ansible_host=192.168.1.21

[appservers]
app1 ansible_host=192.168.1.30

# ─────────────────────────────────────
# VARIABLES FOR ALL HOSTS
# ─────────────────────────────────────
[all:vars]
ansible_user=ec2-user
ansible_ssh_private_key_file=~/.ssh/id_rsa
ansible_python_interpreter=/usr/bin/python3
ansible_host_key_checking=False

# ─────────────────────────────────────
# VARIABLES FOR SPECIFIC GROUP
# ─────────────────────────────────────
[webservers:vars]
http_port=80
max_connections=200

[dbservers:vars]
db_port=3306
```

#### Format 2: YAML

```yaml
all:
  children:
    webservers:
      hosts:
        web1:
          ansible_host: 192.168.1.10
        web2:
          ansible_host: 192.168.1.11
    dbservers:
      hosts:
        db1:
          ansible_host: 192.168.1.20
  vars:
    ansible_user: ec2-user
```

> **Recommendation:** Use INI format when starting out. It's simpler to read and write.

***

### Understanding Groups

Groups let you **target specific sets of servers** when running playbooks.

```
ALL HOSTS
├── webservers
│   ├── web1 (192.168.1.10)
│   └── web2 (192.168.1.11)
├── dbservers
│   ├── db1 (192.168.1.20)
│   └── db2 (192.168.1.21)
└── appservers
    └── app1 (192.168.1.30)
```

**Targeting examples:**

```bash
ansible all          → runs on every host
ansible webservers   → runs only on web1 and web2
ansible dbservers    → runs only on db1 and db2
ansible web1         → runs only on web1
```

***

### Host Variables in Inventory

You can define variables per host directly in the inventory:

```ini
[webservers]
web1 ansible_host=192.168.1.10 ansible_user=ubuntu http_port=80
web2 ansible_host=192.168.1.11 ansible_user=ec2-user http_port=8080
```

**Built-in Ansible variables for hosts:**

| Variable                       | Purpose                      | Example              |
| ------------------------------ | ---------------------------- | -------------------- |
| `ansible_host`                 | IP or hostname to connect to | `192.168.1.10`       |
| `ansible_user`                 | SSH username                 | `ec2-user`, `ubuntu` |
| `ansible_port`                 | SSH port                     | `22`, `2222`         |
| `ansible_ssh_private_key_file` | Path to private key          | `~/.ssh/id_rsa`      |
| `ansible_python_interpreter`   | Python path on remote        | `/usr/bin/python3`   |
| `ansible_become`               | Use sudo                     | `true`               |
| `ansible_become_user`          | Which user to become         | `root`               |

***

### Verify Your Inventory

```bash
# See all hosts in a flat list
ansible-inventory -i inventory/hosts.ini --list

# See hosts in a tree structure
ansible-inventory -i inventory/hosts.ini --graph
```

**Example --graph output:**

```
@all:
  |--@ungrouped:
  |  |--192.168.1.5
  |--@webservers:
  |  |--web1
  |  |--web2
  |--@dbservers:
  |  |--db1
  |  |--db2
```

***

### Lab: Create Your Inventory

```bash
# Create a dedicated lab folder
mkdir -p ~/ansible-lab/inventory
cd ~/ansible-lab

# Create inventory file
vim inventory/hosts.ini
```

Add your servers:

```ini
[all:vars]
ansible_user=ec2-user
ansible_ssh_private_key_file=~/.ssh/id_rsa
ansible_python_interpreter=/usr/bin/python3
ansible_host_key_checking=False

[workers]
worker1 ansible_host=<YOUR_WORKER_IP>
```

Test it:

```bash
ansible-inventory -i inventory/hosts.ini --graph
ansible workers -i inventory/hosts.ini -m ping
```
