# Variables

## Variables — Three Ways to Define

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

***

## Using Variables with Jinja2

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
