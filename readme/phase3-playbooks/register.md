# register

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
