# Ansible Configuration File

#### Step 1 — Create the project directory

bash

```bash
mkdir ~/ansible-lab
cd ~/ansible-lab
```

***

#### Step 2 — Create `ansible.cfg`

bash

```bash
vi ansible.cfg
```

Add this content:

```ini
[defaults]
inventory         = ./hosts   ## this is enough, so that on running ansible commands, 
# you don'thave to explicitly call inventory file 

# rest all fields can be created in inventory file as well
remote_user       = ubuntu
private_key_file  = ~/.ssh/ansible
host_key_checking = False
forks             = 10
```

Save and exit → `ESC` → `:wq` → `Enter`

***

#### Step 3 — Create `hosts` inventory file

bash

```bash
vi hosts
```

Add your worker nodes:

```ini
[servers]
worker_node_1 ansible_host=172.31.xx.xx
worker_node_2 ansible_host=172.31.xx.xx

[all:vars]
ansible_user=ubuntu
ansible_ssh_private_key_file=~/.ssh/ansible
```

Save and exit → `ESC` → `:wq` → `Enter`

***

#### Step 4 — Verify structure

bash

```bash
tree ~/ansible-lab
```

Expected:

```
~/ansible-lab/
├── ansible.cfg    ✅
└── hosts          ✅
```

***

#### Step 5 — Test connection

bash

```bash
ansible all -m ping
```

**Expected Output**

```json
worker_node_1 | SUCCESS => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python3"
    },
    "changed": false,
    "ping": "pong"
}
```

**Understanding the Output**

| Field                           | Value              | Meaning                                      |
| ------------------------------- | ------------------ | -------------------------------------------- |
| `worker_node_1`                 | host name          | Which host responded                         |
| `SUCCESS`                       | status             | Task completed without error                 |
| `discovered_interpreter_python` | `/usr/bin/python3` | Ansible auto-detected Python3 on remote host |
| `changed`                       | `false`            | Nothing was modified on the remote host      |
| `ping`                          | `pong`             | Module ran successfully                      |
