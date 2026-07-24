# Ansible Server Setup

A reusable collection of Ansible roles for bootstrapping and hardening Linux servers. Covers base system configuration, user management, SSH hardening, firewall rules, Docker installation, Prometheus metrics collection, and intrusion prevention.

## Requirements

- **Ansible**: 2.15 or higher
- **Target OS**: Ubuntu 20.04 / 22.04 / 24.04
- **Python**: 3.8+ on target hosts
- **Connectivity**: SSH access to target hosts with sudo privileges

## Roles

| Role | Description |
|------|-------------|
| `common` | Updates packages, installs essentials, configures timezone, NTP, hostname, and swap |
| `users` | Creates admin user, sets up SSH keys, configures passwordless sudo |
| `ssh-hardening` | Deploys a hardened sshd_config (key-only auth, port change, rate limiting) |
| `firewall` | Installs and configures UFW with deny-by-default policy |
| `docker` | Installs Docker CE with log rotation and daemon configuration |
| `node-exporter` | Installs Prometheus Node Exporter as a systemd service |
| `fail2ban` | Installs fail2ban with SSH and optional nginx jails |

## Quick Start

1. Clone this repository:

```bash
git clone <repo-url> && cd ansible-server-setup
```

2. Copy and edit the inventory file:

```bash
cp inventory/hosts.example.ini inventory/hosts.ini
# Edit inventory/hosts.ini with your server IPs
```

3. Review and customize variables:

```bash
vim inventory/group_vars/all.yml
```

4. Run the full playbook:

```bash
ansible-playbook playbooks/site.yml
```

5. Or run individual playbooks:

```bash
# Security hardening only
ansible-playbook playbooks/security.yml

# Monitoring setup only
ansible-playbook playbooks/monitoring.yml
```

## Usage Examples

Run against a specific group of hosts:

```bash
ansible-playbook playbooks/site.yml --limit webservers
```

Run specific roles using tags:

```bash
ansible-playbook playbooks/site.yml --tags "docker,monitoring"
```

Dry run (check mode):

```bash
ansible-playbook playbooks/site.yml --check --diff
```

Override variables at runtime:

```bash
ansible-playbook playbooks/site.yml -e "common_hostname=prod-web-01 ssh_port=2222"
```

## Variables Reference

### common

| Variable | Default | Description |
|----------|---------|-------------|
| `common_timezone` | `UTC` | System timezone |
| `common_hostname` | `server01` | System hostname |
| `common_disable_swap` | `false` | Whether to disable swap |
| `common_essential_packages` | `[curl, wget, vim, ...]` | Packages to install |

### users

| Variable | Default | Description |
|----------|---------|-------------|
| `admin_user` | `deploy` | Admin username to create |
| `admin_groups` | `[sudo, docker]` | Groups for the admin user |
| `ssh_authorized_keys` | `[]` | List of SSH public keys |
| `remove_default_users` | `false` | Remove insecure default users |
| `users_to_remove` | `[]` | Users to remove |

### ssh-hardening

| Variable | Default | Description |
|----------|---------|-------------|
| `ssh_port` | `22` | SSH listen port |
| `ssh_allow_users` | `[deploy]` | Users allowed to SSH in |
| `ssh_permit_root` | `false` | Allow root login |
| `ssh_password_auth` | `false` | Allow password authentication |
| `ssh_max_auth_tries` | `3` | Max auth attempts |
| `ssh_login_grace_time` | `30` | Login grace time (seconds) |
| `ssh_client_alive_interval` | `300` | Keepalive interval (seconds) |
| `ssh_client_alive_count_max` | `2` | Keepalive count before disconnect |

### firewall

| Variable | Default | Description |
|----------|---------|-------------|
| `firewall_allowed_ports` | `[ssh_port]` | Ports to allow (always include SSH) |
| `firewall_additional_ports` | `[]` | Extra ports (e.g., `80/tcp`, `443/tcp`) |

### docker

| Variable | Default | Description |
|----------|---------|-------------|
| `docker_edition` | `ce` | Docker edition |
| `docker_users` | `[admin_user]` | Users to add to docker group |
| `docker_log_max_size` | `10m` | Max log file size |
| `docker_log_max_file` | `3` | Max number of log files |

### node-exporter

| Variable | Default | Description |
|----------|---------|-------------|
| `node_exporter_version` | `1.8.2` | Node Exporter version |
| `node_exporter_port` | `9100` | Metrics listen port |
| `node_exporter_textfile_dir` | `/var/lib/node_exporter/textfile_collector` | Textfile collector path |

### fail2ban

| Variable | Default | Description |
|----------|---------|-------------|
| `fail2ban_maxretry` | `5` | Max retries before ban |
| `fail2ban_bantime` | `3600` | Ban duration (seconds) |
| `fail2ban_findtime` | `600` | Failure counting window (seconds) |
| `fail2ban_ssh_enabled` | `true` | Enable SSH jail |
| `fail2ban_nginx_enabled` | `false` | Enable nginx jails |

## Directory Structure

```
ansible-server-setup/
├── ansible.cfg
├── inventory/
│   ├── hosts.example.ini
│   └── group_vars/
│       └── all.yml
├── playbooks/
│   ├── site.yml
│   ├── security.yml
│   └── monitoring.yml
├── roles/
│   ├── common/
│   ├── users/
│   ├── ssh-hardening/
│   ├── firewall/
│   ├── docker/
│   ├── node-exporter/
│   └── fail2ban/
├── .gitignore
└── README.md
```

## Tips

- Always test with `--check --diff` before applying to production.
- Use `--limit` to target specific host groups.
- Keep your real inventory file (`inventory/hosts.ini`) out of version control. Only the example file is tracked.
- Store sensitive variables (passwords, keys) using Ansible Vault:

```bash
ansible-vault encrypt inventory/group_vars/all.yml
ansible-playbook playbooks/site.yml --ask-vault-pass
```

- After changing `ssh_port`, update `firewall_allowed_ports` to match, or you will lock yourself out.

<sub><sup>Originally developed and tested locally during learning. Later organized and pushed to GitHub for portfolio visibility.</sup></sub>
