# loops

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
