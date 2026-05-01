# Understanding Ansible Output

Every task produces output. Learn to read it:

```
PLAY [My playbook name] ************************************

TASK [Gathering Facts] *************************************
ok: [web1]

TASK [Install nginx] ***************************************
changed: [web1]          ← Package was installed (changed state)

TASK [Start nginx] *****************************************
ok: [web1]               ← Service was already running (no change)

TASK [Copy config] *****************************************
skipping: [web1]         ← Task was skipped (condition not met)

TASK [Bad task] ********************************************
fatal: [web1]: FAILED!   ← Task failed

PLAY RECAP *************************************************
web1 : ok=3  changed=1  unreachable=0  failed=0  skipped=1
```

### Output Status Meanings

| Status         | Meaning                                 |
| -------------- | --------------------------------------- |
| `ok`           | Task ran, nothing changed               |
| `changed`      | Task ran and modified something         |
| `skipping`     | Task was skipped (when condition false) |
| `fatal/FAILED` | Task failed with an error               |
| `UNREACHABLE`  | Could not connect to host               |
