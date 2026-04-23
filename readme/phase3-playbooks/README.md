# Ansible Playbooks

***

## What is a Playbook?

A **Playbook** is a YAML file that describes:

* **WHERE** to run (which hosts)
* **WHAT** to do (which tasks)
* **HOW** to do it (which modules and parameters)

Think of it as a **recipe book** for your infrastructure.

```
Playbook
│
├── Play 1  →  Run on webservers
│   ├── Task 1: Install nginx
│   ├── Task 2: Copy config
│   └── Task 3: Start service
│
└── Play 2  →  Run on dbservers
    ├── Task 1: Install MySQL
    └── Task 2: Start MySQL
```

One playbook can have **multiple plays**.\
Each play targets a **different set of hosts**.
