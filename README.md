# Phase 1: Ansible Foundations

---

## 1.1 What is Ansible?

Ansible is an **open-source IT automation tool** maintained by Red Hat.  
It helps you automate repetitive tasks like:

- Installing software on multiple servers
- Configuring applications
- Deploying code
- Managing users and permissions

> **One-line definition:** Write instructions once → run them on 1 or 1000 servers automatically.

---

## 1.2 Why Do We Need Ansible?

### The Problem Without Ansible

Imagine you have 10 servers and need to install Nginx on all of them.

**Manual approach:**
1. SSH into Server 1 → install Nginx → exit
2. SSH into Server 2 → install Nginx → exit
3. Repeat 8 more times...

**Problems with this:**
- Time-consuming (10 servers × 5 minutes = 50 minutes)
- Human error on each server
- No record of what was done
- Not repeatable consistently

### The Ansible Solution

With Ansible:
1. Write one playbook file describing what to do
2. Run one command
3. Ansible installs Nginx on all 10 servers **in parallel** → done in ~5 minutes

```
Your Laptop                 Remote Servers
(Ansible installed)
      |
      |---SSH---> Server 1 (installs Nginx)
      |---SSH---> Server 2 (installs Nginx)
      |---SSH---> Server 3 (installs Nginx)
      |---SSH---> ...
```

---

## 1.3 How Ansible Works (Architecture)

```
┌─────────────────────────────────────────────────┐
│              CONTROL NODE                        │
│    (Your laptop / jump server)                   │
│                                                  │
│   ┌──────────┐  ┌──────────┐  ┌──────────────┐  │
│   │Inventory │  │Playbooks │  │   Ansible    │  │
│   │  File    │  │  (YAML)  │  │  (Installed) │  │
│   └──────────┘  └──────────┘  └──────────────┘  │
└───────────────────────┬─────────────────────────┘
                        │
              SSH Connection
                        │
        ┌───────────────┼───────────────┐
        │               │               │
        ▼               ▼               ▼
┌──────────────┐ ┌──────────────┐ ┌──────────────┐
│   Worker 1   │ │   Worker 2   │ │   Worker 3   │
│  (No Ansible │ │  (No Ansible │ │  (No Ansible │
│   needed)    │ │   needed)    │ │   needed)    │
└──────────────┘ └──────────────┘ └──────────────┘
```

### Key Concept: AGENTLESS

Ansible is **agentless** — you only install Ansible on ONE machine (control node).  
Remote servers need:
- SSH access
- Python installed (usually pre-installed on Linux)

**That's it. Nothing else.**

Compare with other tools:

| Tool    | Architecture   | Agent on servers? |
|---------|----------------|-------------------|
| Ansible | Push-based     | ❌ Not needed      |
| Chef    | Pull-based     | ✅ Required        |
| Puppet  | Pull-based     | ✅ Required        |

> **Push-based** = Control node pushes instructions to servers  
> **Pull-based** = Servers pull instructions from a central server

---

## 1.4 The Three Core Components

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

---

## 1.5 Lab Setup

### What You Need
- 1 Control Node (where Ansible is installed)
- 1+ Worker Nodes (servers to manage)

### Option A: Local Virtual Machines (VirtualBox)
- Install VirtualBox
- Create 2 VMs with CentOS or Ubuntu
- Ensure both can ping each other

### Option B: AWS EC2 (Recommended for Beginners)
- Launch 2 EC2 instances (t2.micro — Free Tier)
- Use same key pair for both
- Same security group with SSH open

### Verify Connectivity First
```bash
# From control node, test if you can reach worker
ping <worker-ip>
```

---

## 1.6 Install Ansible

### On CentOS / RedHat / Amazon Linux

```bash
# Step 1: Update system
sudo dnf update -y

# Step 2: Add EPEL repository (Ansible lives here)
sudo dnf install epel-release -y

# Step 3: Install Ansible
sudo dnf install ansible -y

# Verify installation
ansible --version
```

### On Ubuntu / Debian

```bash
# Step 1: Update package list
sudo apt update

# Step 2: Install Ansible
sudo apt install ansible -y

# Verify installation
ansible --version
```

### Expected Output of `ansible --version`

```
ansible [core 2.14.x]
  config file = /etc/ansible/ansible.cfg
  configured module search path = [...]
  python version = 3.x.x
  jinja version = 3.x.x
```

> If you see this, Ansible is installed correctly ✅

---

## 1.7 SSH Key Setup (Critical Step)

### Why This Matters

When Ansible runs a playbook, it SSH-es into each server automatically.  
If it needs to type a password each time → automation breaks.  
Solution: **Passwordless SSH using key pairs**.

### How Key-Based SSH Works

```
Control Node                    Worker Node
────────────                    ───────────
Private Key  ◄────────────────► Public Key
(stays here,                    (copy this here)
 never share)                   ~/.ssh/authorized_keys
```

- **Private key** = Your identity card (stays on control node)
- **Public key** = The lock (goes on every worker node)

### Step-by-Step Setup

#### Step 1: Generate SSH Key Pair (on Control Node)

```bash
ssh-keygen
```

You'll see prompts:
```
Enter file in which to save the key (/root/.ssh/id_rsa): [Press Enter]
Enter passphrase (empty for no passphrase): [Press Enter]
Enter same passphrase again: [Press Enter]
```

This creates two files:
```
~/.ssh/id_rsa       ← Private key (NEVER share this)
~/.ssh/id_rsa.pub   ← Public key (copy to workers)
```

#### Step 2: Copy Public Key to Worker Node

```bash
ssh-copy-id username@worker-ip

# Example
ssh-copy-id ec2-user@192.168.1.20
```

This command:
1. Asks for the worker's password (last time!)
2. Copies your public key to `/home/username/.ssh/authorized_keys` on the worker

#### Step 3: Test Passwordless Login

```bash
ssh ec2-user@192.168.1.20
```

If it logs in **without asking for a password** → Setup successful ✅

If you have multiple workers, repeat Step 2 for each one.

---

## 1.8 Ansible Configuration File

Ansible's main config file lives at:
```
/etc/ansible/ansible.cfg
```

### Generate a Full Example Config

```bash
ansible-config init --disabled > /etc/ansible/ansible.cfg
```

### Most Important Settings

```ini
[defaults]
# Default inventory file location
inventory = /etc/ansible/hosts

# Default remote user
remote_user = ec2-user

# SSH private key path
private_key_file = ~/.ssh/id_rsa

# Don't check host fingerprint (useful in labs)
host_key_checking = False

# Number of parallel connections
forks = 10
```

---

## 1.9 Your First Ansible Command

### Test Connectivity

```bash
# Ping localhost (your own machine)
ansible localhost -m ping
```

**Expected output:**
```json
localhost | SUCCESS => {
    "changed": false,
    "ping": "pong"
}
```

### Understanding the Output

```
localhost       → which host responded
SUCCESS         → task completed without error
changed: false  → nothing was modified on the server
ping: pong      → module-specific response
```

---

## 1.10 Key Terminology Cheat Sheet

| Term | Meaning |
|------|---------|
| **Control Node** | Machine where Ansible is installed |
| **Worker / Managed Node** | Remote server being managed |
| **Inventory** | File listing all managed servers |
| **Playbook** | YAML file with automation instructions |
| **Task** | A single action in a playbook |
| **Module** | Built-in program that performs a task |
| **Play** | A group of tasks for a set of hosts |
| **Handler** | Task that runs only when notified |
| **Role** | Reusable, organized collection of tasks |
| **Fact** | System info auto-collected by Ansible |
| **Idempotent** | Running same playbook multiple times = same result |

---

## ✅ Phase 1 Checklist

Before moving to Phase 2, confirm:

- [ ] You understand what Ansible is and why it's used
- [ ] You understand agentless architecture
- [ ] You understand push vs pull mechanism
- [ ] You know the 3 core components (Inventory, Playbook, Modules)
- [ ] Ansible is installed on your control node
- [ ] SSH key pair is generated
- [ ] Public key is copied to at least one worker node
- [ ] `ansible localhost -m ping` returns SUCCESS
- [ ] You can SSH into worker node without password

---

## 📝 Phase 1 Practice Exercise

1. Install Ansible on your control node
2. Generate SSH keys
3. Copy public key to your worker node
4. Run `ansible localhost -m ping` → verify SUCCESS
5. Run `ansible <worker-ip> -m ping` → verify SUCCESS
6. Run `ansible <worker-ip> -m command -a "uptime"` → see worker's uptime

If all 6 steps work, you are ready for Phase 2! ✅