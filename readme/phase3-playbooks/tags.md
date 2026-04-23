# Tags

## Tags — Run Specific Tasks

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
