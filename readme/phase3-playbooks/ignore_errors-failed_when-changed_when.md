# ignore\_errors, failed\_when, changed\_when

## `ignore_errors` and `failed_when`

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
