# YAML Syntax Rules

## YAML Syntax Rules

Ansible playbooks are written in YAML.\
YAML is simple but **very strict about formatting**.

### Rule 1: Indentation with Spaces (NOT Tabs)

```yaml
# ✅ CORRECT — 2 spaces
tasks:
  - name: Install nginx
    yum:
      name: nginx
      state: present

# ❌ WRONG — Tab character used
tasks:
	- name: Install nginx    # This will cause an ERROR
```

> Always use **2 spaces** for indentation. Configure your editor to replace tabs with spaces.

***

### Rule 2: Key-Value Pairs

```yaml
name: Loki
age: 28
city: Bangalore
is_active: true
```

***

### Rule 3: Lists (use `-`)

```yaml
fruits:
  - apple
  - banana
  - mango

# Inline list
colors: [red, green, blue]
```

***

### Rule 4: Nested Structure

```yaml
server:
  name: web1
  ip: 192.168.1.10
  specs:
    cpu: 4
    ram: 8GB
    disk: 100GB
```

***

### Rule 5: Strings with Special Characters

```yaml
# ✅ CORRECT — colon in value needs quotes
description: "Install: nginx web server"

# ❌ WRONG — colon without quotes breaks YAML
description: Install: nginx web server

# ✅ CORRECT — both work for booleans
become: true
become: yes
enabled: false
enabled: no
```

***

### Rule 6: Multiline Strings

```yaml
# Literal block (preserves newlines)
message: |
  Line one
  Line two
  Line three

# Folded block (newlines become spaces)
description: >
  This is a very long description
  that wraps across multiple lines
  but becomes one line
```

***

### Rule 7: YAML File Always Starts with `---`

```yaml
---
# This marks the start of a YAML document
- name: My playbook
  hosts: all
```

***

## <mark style="color:red;">Validate YAML Before Running</mark>

```bash
# Check syntax without executing
ansible-playbook --syntax-check playbook.yml

# Dry run (shows what WOULD happen)
ansible-playbook --check playbook.yml
```
