# How Ansible Works

```
┌─────────────────────────────────────────────────┐
│              CONTROL NODE                        │
│    (Your laptop / jump server)                   │
│                                                  │
│   ┌──────────┐  ┌──────────┐  ┌──────────────┐  │
│   │Inventory │  │Playbooks │  │   Ansible    │  │
│   │  File    │  │  (YAML)  │  │  (Installed) │  │
│   └──────────┘  └──────────┘  └──────────────┘  │
└───────────────────────┬─────────────────────────┘
                        │
              SSH Connection
                        │
        ┌───────────────┼───────────────┐
        │               │               │
        ▼               ▼               ▼
┌──────────────┐ ┌──────────────┐ ┌──────────────┐
│   Worker 1   │ │   Worker 2   │ │   Worker 3   │
│  (No Ansible │ │  (No Ansible │ │  (No Ansible │
│   needed)    │ │   needed)    │ │   needed)    │
└──────────────┘ └──────────────┘ └──────────────┘
```

### Key Concept: AGENTLESS

Ansible is **agentless** — you only install Ansible on ONE machine (control node).\
Remote servers need:

* SSH access
* Python installed (usually pre-installed on Linux)

**That's it. Nothing else.**

Compare with other tools:

| Tool    | Architecture | Agent on servers? |
| ------- | ------------ | ----------------- |
| Ansible | Push-based   | ❌ Not needed      |
| Chef    | Pull-based   | ✅ Required        |
| Puppet  | Pull-based   | ✅ Required        |

> **Push-based** = Control node pushes instructions to servers\
> **Pull-based** = Servers pull instructions from a central server
