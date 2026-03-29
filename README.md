# bierchermuesli.hetzner_ddns

[![CI](https://github.com/Bierchermuesli/ansible-role-hetzner_ddns/actions/workflows/ci.yml/badge.svg)](https://github.com/Bierchermuesli/ansible-role-hetzner_ddns/actions/workflows/ci.yml)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](LICENSE)
[![Ansible Galaxy](https://img.shields.io/badge/galaxy-bierchermuesli.hetzner__ddns-blue)](https://galaxy.ansible.com/ui/standalone/roles/bierchermuesli/hetzner_ddns/)

Ansible role to install and configure [filiparag/hetzner_ddns](https://github.com/filiparag/hetzner_ddns) — a lightweight Dynamic DNS daemon for the Hetzner DNS Console.

This role covers the **manual, non-packaged installation** of the shell script and targets any **systemd-based Linux distribution** (Debian, Ubuntu, RHEL/CentOS, Fedora, Arch Linux, etc.).

---

## Requirements

- systemd on the managed host
- `curl` and `jq` — installed automatically by this role
- Ansible 2.9+

---

## Installation

~~**Ansible Galaxy:**~~ 

*(Galayxy is currently broken — namespace issue)*

```bash
ansible-galaxy role install Bierchermuesli.hetzner_ddns
```

**Git:**

```bash
ansible-galaxy install git+https://github.com/Bierchermuesli/ansible-role-hetzner_ddns.git
```

Or pin a specific version:

```bash
ansible-galaxy install git+https://github.com/Bierchermuesli/ansible-role-hetzner_ddns.git,v1.0.0
```

Or add to your `requirements.yml`:

```yaml
roles:
  - name: bierchermuesli.hetzner_ddns
    src: git+https://github.com/Bierchermuesli/ansible-role-hetzner_ddns.git
    version: main
```

---

## Role Variables

### Install source

```yaml
hetzner_ddns_install_source: github_raw   # github_raw | git | file
hetzner_ddns_install_location: ""         # URL or path — auto-computed when empty
hetzner_ddns_version: master              # branch, tag (e.g. v1.0.1), or commit SHA
```

**`github_raw`** (default) — Fetches the raw script file from GitHub **on the Ansible control node**, then copies it to the target. Delegating to localhost avoids IPv6 connectivity issues (with GitHub). The download happens once and is reused across all targets in the same play.

**`git`** — Clones the repository on the control node (also IPv6-safe), then copies the script to the target. Useful when you need a specific commit or want the full repo available locally.

**`file`** — Copies a script file already present on the control node. Useful for air-gapped environments or testing local changes. Requires `hetzner_ddns_install_location` to be set.

The default `install_location` is computed automatically:

| `install_source` | default `install_location` |
|---|---|
| `github_raw` | `https://raw.githubusercontent.com/filiparag/hetzner_ddns/<version>/hetzner_ddns.sh` |
| `git` | `https://github.com/filiparag/hetzner_ddns.git` |
| `file` | *(must be set)* |

Override `install_location` to use a fork or mirror.

### API key

Exactly one of these must be set — providing both or neither will fail the role.

```yaml
hetzner_ddns_api_key: ""            # inline API key
hetzner_ddns_api_key_file: ""       # path to a file containing the API key (on the managed host)
```

### Zones

```yaml
hetzner_ddns_zones:
  - domain: example.com
    records:
      - name: "@"
        type: "A/AAAA"
        ttl: 3600
        interface: eth0
      - name: homelab
        type: A
```

### Daemon settings and defaults

```yaml
# Override daemon settings (all optional)
hetzner_ddns_settings:
  log_file: /var/log/hetzner_ddns.log
  ip_check_cooldown: 30
  request_timeout: 10
  # api_url: https://api.hetzner.cloud/v1
  # ip_url: https://ip.hetzner.com/
  # auto_create_records: false

# Override record defaults (all optional)
hetzner_ddns_defaults:
  type: "A/AAAA"
  ttl: 60
  # interface: eth0
```

### Paths and service

```yaml
hetzner_ddns_config_file: /usr/local/etc/hetzner_ddns.json
hetzner_ddns_bin_dir: /usr/local/bin

hetzner_ddns_manage_service: true   # set to false to skip service management
hetzner_ddns_service_user: daemon
hetzner_ddns_service_group: root
hetzner_ddns_service_enabled: true
hetzner_ddns_service_state: started
```

---

## Dependencies

None.

---

## Example Playbook

Minimal:

```yaml
- hosts: homelab
  roles:
    - role: bierchermuesli.hetzner_ddns
      vars:
        hetzner_ddns_api_key: "yourapikey"
        hetzner_ddns_zones:
          - domain: example.com
            records:
              - name: "@"
```

Full example:

```yaml
- hosts: homelab
  roles:
    - role: bierchermuesli.hetzner_ddns
      vars:
        hetzner_ddns_api_key_file: /run/secrets/hetzner_api_key
        hetzner_ddns_version: v1.0.1
        hetzner_ddns_settings:
          log_file: /var/log/hetzner_ddns.log
          ip_check_cooldown: 60
        hetzner_ddns_defaults:
          type: "A/AAAA"
          ttl: 300
        hetzner_ddns_zones:
          - domain: example.com
            records:
              - name: "@"
                type: "A/AAAA"
                ttl: 3600
                interface: eth0
              - name: homelab
                type: A
          - domain: my-proxy.tld
            records:
              - name: vpn
```

### Air-gapped / IPv6-only environments

If your control node also has trouble reaching GitHub, download the script manually and use `file` mode:

```yaml
vars:
  hetzner_ddns_install_source: file
  hetzner_ddns_install_location: /path/to/hetzner_ddns.sh
```

---

## License

MIT — see [LICENSE](LICENSE)

## Author

[Bierchermuesli](https://github.com/Bierchermuesli)

Upstream project: [filiparag/hetzner_ddns](https://github.com/filiparag/hetzner_ddns)
