# Condition

## Conditionals — `when`

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

***
