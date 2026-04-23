# Playbook Structure

## Anatomy

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

| Field   | Required    | Description              |
| ------- | ----------- | ------------------------ |
| `name`  | Recommended | Description of the play  |
| `hosts` | **Yes**     | Which servers to target  |
| `tasks` | **Yes**     | List of tasks to perform |

### Every Task Must Have

| Field       | Required    | Description                        |
| ----------- | ----------- | ---------------------------------- |
| `name`      | Recommended | What this task does                |
| Module name | **Yes**     | Which module to use (e.g., `yum:`) |

***

## `hosts` — Targeting Servers

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

***

## `become` — Privilege Escalation

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

***

## `gather_facts` — Collecting System Info

By default, Ansible **automatically collects facts** at the start of every play.\
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

***

## `debug` Module — Print Messages

Use `debug` to print information during playbook execution.\
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
