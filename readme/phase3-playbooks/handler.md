# Handler

## Handlers — Conditional Execution

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
