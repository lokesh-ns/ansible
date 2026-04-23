# What is Ansible

Ansible is an **open-source IT automation tool** maintained by Red Hat.\
It helps you automate repetitive tasks like:

* Installing software on multiple servers
* Configuring applications
* Deploying code
* Managing users and permissions

> **One-line definition:** Write instructions once → run them on 1 or 1000 servers automatically.

***

## Why Do We Need Ansible?

### The Problem Without Ansible

Imagine you have 10 servers and need to install Nginx on all of them.

**Manual approach:**

1. SSH into Server 1 → install Nginx → exit
2. SSH into Server 2 → install Nginx → exit
3. Repeat 8 more times...

**Problems with this:**

* Time-consuming (10 servers × 5 minutes = 50 minutes)
* Human error on each server
* No record of what was done
* Not repeatable consistently

### The Ansible Solution

With Ansible:

1. Write one playbook file describing what to do
2. Run one command
3. Ansible installs Nginx on all 10 servers **in parallel** → done in \~5 minutes

```
Your Laptop                 Remote Servers
(Ansible installed)
      |
      |---SSH---> Server 1 (installs Nginx)
      |---SSH---> Server 2 (installs Nginx)
      |---SSH---> Server 3 (installs Nginx)
      |---SSH---> ...
```
