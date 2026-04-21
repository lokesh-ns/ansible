# Ansible Lab Setup

#### 1. **What You Need**

| Role         | Count | Purpose                                         |
| ------------ | ----- | ----------------------------------------------- |
| Control Node | 1     | Where Ansible is installed and commands are run |
| Worker Nodes | 1+    | Servers that Ansible manages                    |

**Option A: AWS EC2 (Recommended for Beginners)**

* Launch **2 EC2 instances** (t2.micro — Free Tier)
* Use the **same key pair** for both
* Same **security group** with SSH port 22 open

***

#### 1.2 Install Ansible (On Control Node Only)

bash

```bash
# Step 1: Update package list
sudo apt update

# Step 2: Install Ansible
sudo apt install ansible -y

# Step 3: Verify installation
ansible --version
```

**Expected Output**

```
ansible [core 2.14.x]
config file = /etc/ansible/ansible.cfg
configured module search path = [...]
python version = 3.x.x
jinja version = 3.x.x
```

If you see this, Ansible is installed correctly ✅

***

#### 2. SSH Key Setup (Critical Step)

**Why This Matters**

When Ansible runs a playbook, it **automatically SSHes into each worker node**. If it needs to type a password each time → automation breaks.

**Solution:** Passwordless SSH using key pairs.

**How Key-Based SSH Works**

```
Control Node                        Worker Node
────────────                        ───────────
Private Key  ◄─────────────────►   Public Key
(stays here,                        (copied here)
 never share)                       ~/.ssh/authorized_keys
```

| Key             | Role                                                    |
| --------------- | ------------------------------------------------------- |
| **Private key** | Your identity card — stays on control node, never share |
| **Public key**  | The lock — copied to every worker node                  |

***

#### Step-by-Step SSH Setup

**Step 1 — Create Project Directory and Generate SSH Key (On Control Node)**

bash

```bash
# Create project directory
mkdir ~/ansible-lab
cd ~/ansible-lab

# Generate SSH key pair with name 'ansible'
ssh-keygen -t rsa -f ~/.ssh/ansible
```

You'll see prompts — just press Enter for all:

```
Enter passphrase (empty for no passphrase): [Press Enter]
Enter same passphrase again:               [Press Enter]
```

This creates two files:

```
~/.ssh/ansible      ← Private key (NEVER share this) 🔒
~/.ssh/ansible.pub  ← Public key  (copy to workers)  🔑
```

***

**Step 2 — Copy Public Key to Worker Nodes**

bash

```bash
# Syntax
ssh-copy-id -i ~/.ssh/ansible.pub ubuntu@<worker-private-ip>

# Example
ssh-copy-id -i ~/.ssh/ansible.pub ubuntu@172.31.22.10
```

This command:

1. Connects to the worker node
2. Copies your public key to `/home/ubuntu/.ssh/authorized_keys` on the worker
3. From now on — **no password needed**

> If you have multiple workers, repeat this for each one.

***

**Step 3 — Test Passwordless Login**

bash

```bash
# Test SSH without password
ssh -i ~/.ssh/ansible ubuntu@172.31.22.10
```

✅ If it logs in **without asking for a password** — setup is successful

❌ If it asks for a password — the key was not copied correctly, repeat Step 2
