# Ansible Foundations

***

***

***

##

***

## 1.8 Ansible Configuration File

Ansible's main config file lives at:

<pre><code><strong>/etc/ansible/ansible.cfg
</strong></code></pre>

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

***

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

***

## 1.10 Key Terminology Cheat Sheet

| Term                      | Meaning                                            |
| ------------------------- | -------------------------------------------------- |
| **Control Node**          | Machine where Ansible is installed                 |
| **Worker / Managed Node** | Remote server being managed                        |
| **Inventory**             | File listing all managed servers                   |
| **Playbook**              | YAML file with automation instructions             |
| **Task**                  | A single action in a playbook                      |
| **Module**                | Built-in program that performs a task              |
| **Play**                  | A group of tasks for a set of hosts                |
| **Handler**               | Task that runs only when notified                  |
| **Role**                  | Reusable, organized collection of tasks            |
| **Fact**                  | System info auto-collected by Ansible              |
| **Idempotent**            | Running same playbook multiple times = same result |

***

## ✅ Phase 1 Checklist

Before moving to Phase 2, confirm:

* [ ] You understand what Ansible is and why it's used
* [ ] You understand agentless architecture
* [ ] You understand push vs pull mechanism
* [ ] You know the 3 core components (Inventory, Playbook, Modules)
* [ ] Ansible is installed on your control node
* [ ] SSH key pair is generated
* [ ] Public key is copied to at least one worker node
* [ ] `ansible localhost -m ping` returns SUCCESS
* [ ] You can SSH into worker node without password

***

## 📝 Phase 1 Practice Exercise

1. Install Ansible on your control node
2. Generate SSH keys
3. Copy public key to your worker node
4. Run `ansible localhost -m ping` → verify SUCCESS
5. Run `ansible <worker-ip> -m ping` → verify SUCCESS
6. Run `ansible <worker-ip> -m command -a "uptime"` → see worker's uptime

If all 6 steps work, you are ready for Phase 2! ✅
