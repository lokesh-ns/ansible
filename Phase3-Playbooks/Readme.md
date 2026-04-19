# Phase 3: Playbooks — Deep Dive

---

## 3.1 What is a Playbook?

A **Playbook** is a YAML file that describes:
- **WHERE** to run (which hosts)
- **WHAT** to do (which tasks)
- **HOW** to do it (which modules and parameters)

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

One playbook can have **multiple plays**.  
Each play targets a **different set of hosts**.

---

## 3.2 Playbook Structure — Anatomy

```yaml
---
# ─────────────────────────────────────────────
# PLAY starts here
# ─────────────────────────────────────────────
- name: Setup Web Servers          # Play name (descriptive)
  hosts: webservers                # Which hosts to target
  become: true                     # Use sudo for all tasks
  gather_facts: true               # Collect system info (default: true)

  # ─────────────────────────────
  # VARIABLES (optional)
  # ─────────────────────────────
  vars:
    app_name: nginx
    app_port: 80

  # ─────────────────────────────
  # TASKS (required)
  # ─────────────────────────────
  tasks:
    - name: Install nginx          # Task name (descriptive)
      yum:                         # Module name
        name: "{{ app_name }}"     # Module parameter
        state: present             # Module parameter

    - name: Start nginx
      service:
        name: "{{ app_name }}"
        state: started
        enabled: true

  # ─────────────────────────────
  # HANDLERS (optional)
  # ─────────────────────────────
  handlers:
    - name: Restart nginx
      service:
        name: nginx
        state: restarted
```

### Every Play Must Have

| Field | Required | Description |
|-------|----------|-------------|
| `name` | Recommended | Description of the play |
| `hosts` | **Yes** | Which servers to target |
| `tasks` | **Yes** | List of tasks to perform |

### Every Task Must Have

| Field | Required | Description |
|-------|----------|-------------|
| `name` | Recommended | What this task does |
| Module name | **Yes** | Which module to use (e.g., `yum:`) |

---

## 3.3 `hosts` — Targeting Servers

The `hosts` field controls which servers a play runs on.

```yaml
# Run on ALL hosts in inventory
hosts: all

# Run on a specific group
hosts: webservers

# Run on multiple groups
hosts: webservers,dbservers

# Run on a single host by name
hosts: web1

# Run on localhost (control node itself)
hosts: localhost

# Run on hosts matching a pattern
hosts: web*          # Matches web1, web2, webserver, etc.

# Exclude a group
hosts: all:!dbservers   # All hosts except dbservers
```

---

## 3.4 `become` — Privilege Escalation

Many tasks need root/sudo access. Use `become`.

```yaml
# ─────────────────────────────
# At play level (applies to ALL tasks)
# ─────────────────────────────
- name: My play
  hosts: all
  become: true               # All tasks run as sudo

# ─────────────────────────────
# At task level (applies to ONE task)
# ─────────────────────────────
- name: My play
  hosts: all

  tasks:
    - name: This runs as normal user
      command: whoami

    - name: This runs as root
      yum:
        name: nginx
        state: present
      become: true            # Only this task uses sudo

    - name: This runs as specific user
      command: /opt/myapp/start.sh
      become: true
      become_user: appuser    # Become this user instead of root
```

### Running Playbook as Non-Root User

```bash
# Prompt for sudo password
ansible-playbook -i inventory/hosts.ini playbook.yml --ask-become-pass

# Or use -K shorthand
ansible-playbook -i inventory/hosts.ini playbook.yml -K
```

---

## 3.5 `gather_facts` — Collecting System Info

By default, Ansible **automatically collects facts** at the start of every play.  
This is the `Gathering Facts` task you see in output.

```yaml
# Default — facts collected
- name: My play
  hosts: all
  gather_facts: true    # Default, don't need to specify

# Disable for faster execution (when you don't need facts)
- name: Fast play
  hosts: all
  gather_facts: false
```

**When to disable:** If you have hundreds of servers and don't use any `ansible_*` variables, disabling speeds up execution significantly.

---

## 3.6 `debug` Module — Print Messages

Use `debug` to print information during playbook execution.  
Very useful for troubleshooting.

```yaml
tasks:
  # ─────────────────────────────
  # Print a simple message
  # ─────────────────────────────
  - name: Print hello
    debug:
      msg: "Hello from {{ inventory_hostname }}"

  # ─────────────────────────────
  # Print a variable value
  # ─────────────────────────────
  - name: Print OS info
    debug:
      msg: "Running {{ ansible_distribution }} {{ ansible_distribution_version }}"

  # ─────────────────────────────
  # Print entire variable contents
  # ─────────────────────────────
  - name: Print all facts
    debug:
      var: ansible_default_ipv4

  # ─────────────────────────────
  # Print only when verbose mode
  # ─────────────────────────────
  - name: Debug message (shows only with -v flag)
    debug:
      msg: "This is a debug message"
      verbosity: 1          # Only shows with -v or higher
```

---

## 3.7 Variables — Three Ways to Define

### Way 1: In the Playbook (`vars` block)

```yaml
---
- name: Variables in playbook
  hosts: all
  become: true

  vars:
    app_name: nginx
    app_port: 80
    app_user: webadmin
    install_dir: /opt/myapp

  tasks:
    - name: Install app
      yum:
        name: "{{ app_name }}"
        state: present

    - name: Create install directory
      file:
        path: "{{ install_dir }}"
        state: directory
        owner: "{{ app_user }}"
        mode: '0755'

    - name: Print app info
      debug:
        msg: "Installing {{ app_name }} on port {{ app_port }}"
```

### Way 2: In a Separate Variables File

```bash
# Create vars directory and file
mkdir -p ~/ansible-lab/vars
vim ~/ansible-lab/vars/app_config.yml
```

```yaml
# vars/app_config.yml
app_name: nginx
app_port: 80
app_user: webadmin
install_dir: /opt/myapp
db_host: localhost
db_port: 3306
```

```yaml
# In your playbook
---
- name: Using external vars file
  hosts: all
  become: true

  vars_files:
    - vars/app_config.yml       # Load variables from file
    - vars/db_config.yml        # Load multiple files

  tasks:
    - name: Install app
      yum:
        name: "{{ app_name }}"
        state: present
```

### Way 3: Runtime Variables (Command Line)

```bash
# Pass single variable
ansible-playbook playbook.yml --extra-vars "app_name=httpd"

# Pass multiple variables
ansible-playbook playbook.yml --extra-vars "app_name=httpd app_port=8080"

# Pass as JSON
ansible-playbook playbook.yml --extra-vars '{"app_name": "httpd", "app_port": 8080}'

# Pass from a file
ansible-playbook playbook.yml --extra-vars "@vars/prod_config.yml"
```

### Variable Precedence (Lowest to Highest)

```
role defaults          ← Lowest priority (easiest to override)
inventory vars
playbook vars
vars_files
vars
task vars
--extra-vars           ← Highest priority (always wins)
```

> **Remember:** `--extra-vars` always overrides everything else.

---

## 3.8 Using Variables with Jinja2

Variables are accessed using **double curly braces**: `{{ variable_name }}`

```yaml
vars:
  server_name: myapp
  domain: example.com
  port: 8080

tasks:
  # Simple variable usage
  - name: Print server name
    debug:
      msg: "Server: {{ server_name }}"

  # Variable in a string (must quote the whole value)
  - name: Create config
    copy:
      content: "server_name = {{ server_name }}.{{ domain }};"
      dest: /etc/nginx/server.conf

  # Variable in file path
  - name: Create log dir
    file:
      path: "/var/log/{{ server_name }}"
      state: directory

  # Variable with default value (if not defined)
  - name: Print port
    debug:
      msg: "Port: {{ port | default(80) }}"
```

### Jinja2 Filters (Variable Transformations)

```yaml
vars:
  hostname: "WEB-SERVER-01"
  version: "1.9.2"

tasks:
  # Convert to lowercase
  - debug:
      msg: "{{ hostname | lower }}"         # web-server-01

  # Convert to uppercase
  - debug:
      msg: "{{ hostname | upper }}"         # WEB-SERVER-01

  # Replace characters
  - debug:
      msg: "{{ hostname | replace('-', '_') }}"   # WEB_SERVER_01

  # Get list length
  - debug:
      msg: "{{ packages | length }}"

  # Default value if undefined
  - debug:
      msg: "{{ undefined_var | default('not set') }}"

  # Hash a password
  - user:
      name: admin
      password: "{{ 'MyPass123' | password_hash('sha512') }}"
```

---

## 3.9 Conditionals — `when`

Run a task **only when a condition is true**.

### Basic Syntax

```yaml
- name: Task name
  module:
    parameter: value
  when: condition
```

### Condition Examples

```yaml
vars:
  env: production
  app_version: "2.0"
  enable_backup: true

tasks:
  # ─────────────────────────────
  # Simple comparison
  # ─────────────────────────────
  - name: Only in production
    debug:
      msg: "Running in production"
    when: env == "production"

  # ─────────────────────────────
  # Not equal
  # ─────────────────────────────
  - name: Not in development
    debug:
      msg: "Not dev environment"
    when: env != "development"

  # ─────────────────────────────
  # Boolean check
  # ─────────────────────────────
  - name: Run backup
    command: /usr/local/bin/backup.sh
    when: enable_backup

  # ─────────────────────────────
  # Using Ansible facts
  # ─────────────────────────────
  - name: Install on RedHat only
    yum:
      name: httpd
      state: present
    when: ansible_os_family == "RedHat"

  - name: Install on Ubuntu only
    apt:
      name: apache2
      state: present
    when: ansible_os_family == "Debian"

  # ─────────────────────────────
  # Multiple conditions (AND)
  # ─────────────────────────────
  - name: Install on RedHat 8 only
    yum:
      name: httpd
      state: present
    when:
      - ansible_os_family == "RedHat"
      - ansible_distribution_major_version == "8"

  # ─────────────────────────────
  # Multiple conditions (OR)
  # ─────────────────────────────
  - name: Install on CentOS or RHEL
    yum:
      name: httpd
      state: present
    when: >
      ansible_distribution == "CentOS" or
      ansible_distribution == "RedHat"

  # ─────────────────────────────
  # Check if variable is defined
  # ─────────────────────────────
  - name: Use custom port if defined
    debug:
      msg: "Using port {{ custom_port }}"
    when: custom_port is defined

  # ─────────────────────────────
  # Check if variable is not defined
  # ─────────────────────────────
  - name: Use default port
    debug:
      msg: "Using default port 80"
    when: custom_port is not defined
```

---

## 3.10 Loops — Repeat Tasks

Run the same task multiple times with different values.

### Basic Loop

```yaml
tasks:
  - name: Install multiple packages
    yum:
      name: "{{ item }}"          # item = current value in loop
      state: present
    loop:
      - nginx
      - git
      - vim
      - curl
      - wget
```

### Loop with Dictionary Items

```yaml
tasks:
  - name: Create multiple users
    user:
      name: "{{ item.name }}"
      groups: "{{ item.groups }}"
      shell: "{{ item.shell }}"
    loop:
      - { name: "alice",   groups: "wheel",  shell: "/bin/bash" }
      - { name: "bob",     groups: "nginx",  shell: "/bin/bash" }
      - { name: "charlie", groups: "docker", shell: "/bin/zsh"  }
```

### Loop with Variable List

```yaml
vars:
  packages:
    - nginx
    - git
    - vim
    - tree

  users:
    - alice
    - bob
    - charlie

tasks:
  - name: Install packages from variable
    yum:
      name: "{{ item }}"
      state: present
    loop: "{{ packages }}"

  - name: Create users from variable
    user:
      name: "{{ item }}"
      state: present
    loop: "{{ users }}"
```

### Loop with Index

```yaml
tasks:
  - name: Print item with index
    debug:
      msg: "Item {{ ansible_loop.index }}: {{ item }}"
    loop:
      - apple
      - banana
      - cherry
    loop_control:
      index_var: my_index     # Custom index variable name
```

### Loop + Conditional

```yaml
vars:
  services:
    - { name: nginx,  enabled: true  }
    - { name: httpd,  enabled: false }
    - { name: mysql,  enabled: true  }

tasks:
  - name: Start only enabled services
    service:
      name: "{{ item.name }}"
      state: started
    loop: "{{ services }}"
    when: item.enabled        # Only runs when enabled is true
```

---

## 3.11 Handlers — Conditional Execution

A **handler** is a task that **only runs when notified by another task**.

### Why Use Handlers?

Without handlers:
```yaml
tasks:
  - name: Update nginx config
    copy:
      src: nginx.conf
      dest: /etc/nginx/nginx.conf

  - name: Restart nginx              # Always restarts, even if config didn't change
    service:
      name: nginx
      state: restarted
```

With handlers:
```yaml
tasks:
  - name: Update nginx config
    copy:
      src: nginx.conf
      dest: /etc/nginx/nginx.conf
    notify: Restart nginx            # Only notifies if config CHANGED

handlers:
  - name: Restart nginx
    service:
      name: nginx
      state: restarted               # Only runs if notified
```

### Key Rules for Handlers

1. **Handlers run at the END** of a play, not immediately when notified
2. **Handlers only run ONCE** even if notified multiple times
3. **Handler is identified by its `name`** — must match exactly
4. **Handler only runs if the notifying task had `changed` status**

```yaml
---
- name: Nginx Setup with Handlers
  hosts: webservers
  become: true

  tasks:
    - name: Install nginx
      yum:
        name: nginx
        state: present
      notify: Start nginx            # Notify if nginx was just installed

    - name: Update nginx config
      copy:
        src: nginx.conf
        dest: /etc/nginx/nginx.conf
      notify:
        - Validate nginx config      # Notify multiple handlers
        - Restart nginx

    - name: Update nginx SSL cert
      copy:
        src: ssl/cert.pem
        dest: /etc/nginx/ssl/cert.pem
      notify: Restart nginx          # Same handler, will only run ONCE

  handlers:
    - name: Validate nginx config
      command: nginx -t

    - name: Restart nginx
      service:
        name: nginx
        state: restarted

    - name: Start nginx
      service:
        name: nginx
        state: started
        enabled: true
```

### Force Handlers to Run Immediately

```yaml
tasks:
  - name: Update config
    copy:
      src: app.conf
      dest: /etc/app/app.conf
    notify: Restart app

  # Force handlers to run NOW instead of at end
  - name: Flush handlers
    meta: flush_handlers

  - name: Check app is running
    command: systemctl status myapp
```

---

## 3.12 Tags — Run Specific Tasks

Tags let you **run or skip specific tasks** without creating separate playbooks.

### Adding Tags

```yaml
---
- name: Full server setup
  hosts: all
  become: true

  tasks:
    - name: Install nginx
      yum:
        name: nginx
        state: present
      tags:
        - install
        - nginx

    - name: Copy nginx config
      copy:
        src: nginx.conf
        dest: /etc/nginx/nginx.conf
      tags:
        - configure
        - nginx

    - name: Start nginx
      service:
        name: nginx
        state: started
      tags:
        - service
        - nginx

    - name: Install MySQL
      yum:
        name: mysql-server
        state: present
      tags:
        - install
        - mysql

    - name: Create app user
      user:
        name: appuser
        state: present
      tags:
        - users
```

### Running with Tags

```bash
# See all available tags
ansible-playbook -i inventory/hosts.ini playbook.yml --list-tags

# Run only tasks tagged 'install'
ansible-playbook -i inventory/hosts.ini playbook.yml --tags install

# Run only tasks tagged 'nginx'
ansible-playbook -i inventory/hosts.ini playbook.yml --tags nginx

# Run tasks with multiple tags
ansible-playbook -i inventory/hosts.ini playbook.yml --tags "install,users"

# Skip tasks tagged 'mysql'
ansible-playbook -i inventory/hosts.ini playbook.yml --skip-tags mysql

# Skip multiple tags
ansible-playbook -i inventory/hosts.ini playbook.yml --skip-tags "mysql,users"
```

### Special Built-in Tags

```bash
# Run ALL tasks (including those with never tag)
ansible-playbook playbook.yml --tags all

# Run ALL tasks (default behavior)
ansible-playbook playbook.yml

# Tasks tagged 'always' always run, regardless of --tags
- name: Always collect facts
  setup:
  tags: always

# Tasks tagged 'never' only run when explicitly called
- name: Debug task
  debug:
    msg: "Debug info"
  tags: never
```

---

## 3.13 `register` — Capture Task Output

Capture the output of a task into a variable for later use.

```yaml
tasks:
  # ─────────────────────────────
  # Register command output
  # ─────────────────────────────
  - name: Check disk space
    command: df -h /
    register: disk_output          # Save output in this variable

  - name: Print disk space
    debug:
      msg: "{{ disk_output.stdout }}"

  # ─────────────────────────────
  # Register and check return code
  # ─────────────────────────────
  - name: Check if nginx is running
    command: systemctl is-active nginx
    register: nginx_status
    ignore_errors: true            # Don't fail if nginx is stopped

  - name: Start nginx if not running
    service:
      name: nginx
      state: started
    when: nginx_status.rc != 0    # rc = return code (0 = success)

  # ─────────────────────────────
  # Register and check output
  # ─────────────────────────────
  - name: Get running processes
    shell: ps aux | grep nginx
    register: process_list

  - name: Show nginx processes
    debug:
      msg: "{{ process_list.stdout_lines }}"   # List of output lines
```

### Registered Variable Fields

```
variable.stdout         → Standard output as string
variable.stderr         → Standard error as string
variable.stdout_lines   → Standard output as list of lines
variable.rc             → Return code (0 = success)
variable.changed        → Whether the task changed anything
variable.failed         → Whether the task failed
```

---

## 3.14 `ignore_errors` and `failed_when`

### `ignore_errors` — Continue Even if Task Fails

```yaml
tasks:
  - name: Remove file if it exists
    file:
      path: /tmp/oldfile.txt
      state: absent
    ignore_errors: true           # Continue even if file doesn't exist

  - name: This task always runs
    debug:
      msg: "Continuing after possible error above"
```

### `failed_when` — Custom Failure Conditions

```yaml
tasks:
  - name: Check service status
    command: systemctl status nginx
    register: service_status
    failed_when: "'active' not in service_status.stdout"
    # Task fails if 'active' is NOT in the output

  - name: Run script
    shell: /opt/scripts/deploy.sh
    register: deploy_result
    failed_when:
      - deploy_result.rc != 0
      - "'ERROR' in deploy_result.stdout"
    # Task fails if return code != 0 AND output contains ERROR
```

### `changed_when` — Control When Task Reports as Changed

```yaml
tasks:
  - name: Check if update needed
    command: yum check-update nginx
    register: update_check
    changed_when: update_check.rc == 100    # yum returns 100 when updates available
    failed_when: update_check.rc not in [0, 100]
```

---

## 3.15 `block` — Group Tasks Together

Group multiple tasks and handle errors for the whole group.

```yaml
tasks:
  - name: Install and configure nginx
    block:
      - name: Install nginx
        yum:
          name: nginx
          state: present

      - name: Copy config
        copy:
          src: nginx.conf
          dest: /etc/nginx/nginx.conf

      - name: Start nginx
        service:
          name: nginx
          state: started

    rescue:
      # Runs if ANY task in the block fails
      - name: Log failure
        debug:
          msg: "Nginx setup failed! Check the configuration."

      - name: Ensure nginx is stopped
        service:
          name: nginx
          state: stopped
        ignore_errors: true

    always:
      # ALWAYS runs, whether block succeeded or failed
      - name: Check nginx status
        command: systemctl status nginx
        ignore_errors: true
```

### Block with `when` Condition

```yaml
- name: RedHat specific setup
  block:
    - name: Install httpd
      yum:
        name: httpd
        state: present

    - name: Start httpd
      service:
        name: httpd
        state: started
  when: ansible_os_family == "RedHat"
```

---

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

---

## 3.17 Complete Real-World Playbook Example

This example ties together everything learned so far:

```yaml
---
- name: Full Application Server Setup
  hosts: appservers
  become: true
  gather_facts: true

  # ─────────────────────────────
  # Variables
  # ─────────────────────────────
  vars:
    app_name: mywebapp
    app_user: webadmin
    app_dir: /opt/mywebapp
    app_port: 8080
    packages:
      - nginx
      - git
      - curl
      - python3

  # ─────────────────────────────
  # Tasks
  # ─────────────────────────────
  tasks:

    # Print server info
    - name: Print server information
      debug:
        msg: >
          Setting up {{ app_name }} on
          {{ ansible_hostname }} ({{ ansible_distribution }})

    # Install packages based on OS
    - name: Install packages on RedHat
      yum:
        name: "{{ packages }}"
        state: present
      when: ansible_os_family == "RedHat"
      tags:
        - install
        - packages

    - name: Install packages on Debian
      apt:
        name: "{{ packages }}"
        state: present
        update_cache: true
      when: ansible_os_family == "Debian"
      tags:
        - install
        - packages

    # Create application user
    - name: Create application user
      user:
        name: "{{ app_user }}"
        comment: "Web Application User"
        shell: /bin/bash
        create_home: true
      tags:
        - users

    # Create app directory
    - name: Create application directory
      file:
        path: "{{ app_dir }}"
        state: directory
        owner: "{{ app_user }}"
        group: "{{ app_user }}"
        mode: '0755'
      tags:
        - configure

    # Create app config file
    - name: Deploy application config
      copy:
        content: |
          # Application Configuration
          # Generated by Ansible on {{ ansible_date_time.date }}
          APP_NAME={{ app_name }}
          APP_PORT={{ app_port }}
          APP_USER={{ app_user }}
          SERVER={{ ansible_hostname }}
          OS={{ ansible_distribution }}
        dest: "{{ app_dir }}/app.conf"
        owner: "{{ app_user }}"
        mode: '0640'
      notify: Restart nginx
      tags:
        - configure

    # Start and enable nginx
    - name: Start and enable nginx
      service:
        name: nginx
        state: started
        enabled: true
      tags:
        - service

    # Configure firewall
    - name: Open app port in firewall
      firewalld:
        port: "{{ app_port }}/tcp"
        state: enabled
        permanent: true
      notify: Reload firewall
      when: ansible_os_family == "RedHat"
      tags:
        - firewall

    # Schedule health check
    - name: Schedule health check
      cron:
        name: "{{ app_name }} health check"
        minute: "*/15"
        user: "{{ app_user }}"
        job: "curl -s http://localhost:{{ app_port }}/health >> /var/log/health.log 2>&1"
      tags:
        - cron

    # Verify setup
    - name: Verify nginx is running
      command: systemctl is-active nginx
      register: nginx_check
      changed_when: false

    - name: Print verification result
      debug:
        msg: "Nginx status: {{ nginx_check.stdout }}"

  # ─────────────────────────────
  # Handlers
  # ─────────────────────────────
  handlers:
    - name: Restart nginx
      service:
        name: nginx
        state: restarted

    - name: Reload firewall
      service:
        name: firewalld
        state: reloaded
```

### Run This Playbook

```bash
# Full run
ansible-playbook -i inventory/hosts.ini setup_app.yml

# Dry run (no changes made)
ansible-playbook -i inventory/hosts.ini setup_app.yml --check

# Only install tasks
ansible-playbook -i inventory/hosts.ini setup_app.yml --tags install

# Skip cron tasks
ansible-playbook -i inventory/hosts.ini setup_app.yml --skip-tags cron

# Verbose output
ansible-playbook -i inventory/hosts.ini setup_app.yml -v

# With sudo password prompt
ansible-playbook -i inventory/hosts.ini setup_app.yml -K
```

---

## 3.18 Playbook Best Practices

### 1. Always Name Your Plays and Tasks

```yaml
# ✅ GOOD — descriptive names
- name: Install nginx web server
  yum:
    name: nginx
    state: present

# ❌ BAD — no name (hard to debug)
- yum:
    name: nginx
    state: present
```

### 2. Use Variables Instead of Hardcoding

```yaml
# ✅ GOOD — flexible
vars:
  app_name: nginx

tasks:
  - name: Install {{ app_name }}
    yum:
      name: "{{ app_name }}"

# ❌ BAD — hardcoded
tasks:
  - name: Install nginx
    yum:
      name: nginx
```

### 3. Validate Before Running

```bash
# Always check syntax first
ansible-playbook --syntax-check playbook.yml

# Then do a dry run
ansible-playbook --check playbook.yml

# Then run for real
ansible-playbook playbook.yml
```

### 4. Use Tags Consistently

```yaml
# Good tagging strategy
- name: Install packages
  yum: ...
  tags: [install, packages]

- name: Configure app
  copy: ...
  tags: [configure]

- name: Start service
  service: ...
  tags: [service]
```

### 5. Organize Your Project

```
ansible-lab/
├── inventory/
│   ├── hosts.ini
│   └── group_vars/
│       ├── all.yml
│       └── webservers.yml
├── vars/
│   └── app_config.yml
├── files/
│   └── nginx.conf
├── playbooks/
│   ├── setup_web.yml
│   ├── setup_db.yml
│   └── common.yml
└── README.md
```

### 6. Use `group_vars` for Group-Specific Variables

```bash
mkdir -p inventory/group_vars
```

```yaml
# inventory/group_vars/webservers.yml
http_port: 80
max_connections: 1000
nginx_user: nginx

# inventory/group_vars/dbservers.yml
db_port: 3306
db_name: myapp
db_user: appuser

# inventory/group_vars/all.yml   (applies to all hosts)
ansible_user: ec2-user
ansible_python_interpreter: /usr/bin/python3
timezone: Asia/Kolkata
```

These files are automatically loaded — no need to specify them in playbooks.

---

## ✅ Phase 3 Checklist

Before moving to Phase 4, confirm you can:

- [ ] Understand playbook structure (play, tasks, handlers)
- [ ] Target specific hosts and groups with `hosts:`
- [ ] Use `become` at play level and task level
- [ ] Disable `gather_facts` when not needed
- [ ] Use `debug` module to print messages and variables
- [ ] Define variables three ways (playbook, vars file, runtime)
- [ ] Use Jinja2 `{{ variable }}` syntax correctly
- [ ] Apply basic Jinja2 filters (lower, upper, default)
- [ ] Write `when` conditions with single and multiple checks
- [ ] Use facts in `when` conditions for OS-specific tasks
- [ ] Use `loop` with a list, variable, and dictionary
- [ ] Combine `loop` with `when`
- [ ] Create handlers and trigger them with `notify`
- [ ] Understand that handlers run once at end of play
- [ ] Add tags to tasks and run playbooks with `--tags`
- [ ] Use `register` to capture task output
- [ ] Use `ignore_errors`, `failed_when`, `changed_when`
- [ ] Write a multi-play playbook
- [ ] Run playbooks with `--check`, `--syntax-check`, `-v`
- [ ] Organize project with `group_vars`

---

## 📝 Phase 3 Practice Exercise

Build a playbook `full_setup.yml` that does:

1. **Play 1 — Common Setup (all hosts):**
   - Install common packages: `vim`, `git`, `curl`, `tree`, `wget`
   - Create a user `devops` with shell `/bin/bash` and sudo access
   - Print OS distribution and hostname using `debug`

2. **Play 2 — Web Server Setup (webservers group):**
   - Install `nginx` (use `when` to handle RedHat vs Debian)
   - Copy a custom `index.html` to `/usr/share/nginx/html/`
   - Start and enable nginx
   - Open port 80 in firewall
   - Use a handler to restart nginx when config changes

3. **Play 3 — Database Setup (dbservers group):**
   - Install `mysql-server`
   - Create a cron job for daily DB backup at 3 AM
   - Start and enable MySQL

**Requirements:**
- All tasks must have descriptive `name`
- Use at least 3 variables
- Use `when` condition for OS-specific tasks
- Use `loop` for package installation
- Add tags to all tasks
- Use at least one handler

Run it with:
```bash
ansible-playbook -i inventory/hosts.ini full_setup.yml --syntax-check
ansible-playbook -i inventory/hosts.ini full_setup.yml --check
ansible-playbook -i inventory/hosts.ini full_setup.yml -v
```

Fix all errors until `failed=0` ✅sss