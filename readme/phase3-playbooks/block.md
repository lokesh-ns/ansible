# block

## `block` — Group Tasks Together

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

***
