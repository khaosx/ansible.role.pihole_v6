# Role: pihole_v6

## Description

Installs and configures Pi-hole v6 on Debian/Ubuntu hosts, with optional Unbound, TLS via Let's Encrypt DNS-01 (Cloudflare), API-driven adlist management, and optional DNS smoke tests.

## Requirements

- Ansible >= 2.14
- Target OS: Ubuntu, Debian
- Role dependency available when firewall management is enabled: `ufw_profiles`

## Privilege Escalation

requires_become: true

## Role Variables

| Variable | Default | Required | Description |
|---|---|---|---|
| `pihole_v6_install_unbound` | `true` | No | Enable Unbound installation and configuration. |
| `pihole_v6_enable_tls` | `true` | No | Enable TLS issuance and PEM deployment. |
| `pihole_v6_enable_tests` | `false` | No | Run DNS smoke tests. |
| `pihole_v6_setup_dnssec` | `false` | No | Include DNSSEC smoke tests. |
| `pihole_v6_ufw_enable` | `true` | No | Manage UFW exposure through `ufw_profiles`. |
| `pihole_v6_ufw_profile` | see defaults | No | UFW application profile definition for Pi-hole services. |
| `pihole_v6_install_script_url` | `https://install.pi-hole.net` | No | Pi-hole installer URL. |
| `pihole_v6_listen_address` | `127.0.0.1` | No | Unbound bind address used by Pi-hole upstream DNS. |
| `pihole_v6_listen_port` | `5335` | No | Unbound bind port used by Pi-hole upstream DNS. |
| `pihole_v6_unbound_root_hints_url` | `https://www.internic.net/domain/named.cache` | No | Root hints source for Unbound. |
| `pihole_v6_dns_interface` | `eth0` | No | Value for `dns.interface` in `pihole.toml`. |
| `pihole_v6_dns_listening_mode` | `BIND` | No | Value for `dns.listeningMode` in `pihole.toml`. |
| `pihole_v6_dns_rev_servers` | `[]` | No | Value for `dns.revServers` in `pihole.toml`. |
| `pihole_v6_admin_password` | unset | Yes when adlist API management is enabled | Pi-hole admin password for API login. |
| `pihole_v6_port` | `80` | No | Pi-hole web/API port. |
| `pihole_v6_api_scheme` | `http` | No | Pi-hole API scheme. |
| `pihole_v6_api_host` | `{{ ansible_host \\| default(inventory_hostname) }}` | No | Pi-hole API hostname. |
| `pihole_v6_api_port` | `{{ pihole_v6_port }}` | No | Pi-hole API port. |
| `pihole_v6_api_validate_certs` | `false` | No | Enable TLS cert validation for API requests. |
| `pihole_v6_api_timeout` | `30` | No | API timeout in seconds. |
| `pihole_v6_api_wait_timeout` | `60` | No | API wait timeout in seconds. |
| `pihole_v6_api_base_url` | derived | No | Pi-hole API base URL. |
| `pihole_v6_toml_force` | `false` | No | Force overwrite of `pihole.toml`. |
| `pihole_v6_local_domain` | `local` | No | Local DNS domain. |
| `pihole_v6_web_domain` | `{{ inventory_hostname }}.{{ pihole_v6_local_domain }}` | No | Web UI domain configured in `pihole.toml`. |
| `pihole_v6_webserver_acl` | `""` | No | Web UI ACL string. |
| `pihole_v6_webserver_ports` | derived | No | Web UI bind ports string. |
| `pihole_v6_webserver_pwhash` | unset | Yes | Pi-hole web/API password hash. |
| `pihole_v6_webserver_app_pwhash` | unset | Yes | Pi-hole app password hash. |
| `pihole_v6_adlists_present` | see defaults | No | Desired blocklists to enforce through API. |
| `pihole_v6_default_list_address` | `https://raw.githubusercontent.com/StevenBlack/hosts/master/hosts` | No | Default list URL to delete if enabled. |
| `pihole_v6_delete_default_list` | `true` | No | Delete Pi-hole default adlist when present. |
| `pihole_v6_trigger_gravity` | `true` | No | Trigger gravity rebuild after list changes. |
| `pihole_v6_tls_domain` | `{{ inventory_hostname }}.{{ pihole_v6_local_domain }}` | No | Domain for certificate issuance. |
| `pihole_v6_tls_email` | `root@localhost` | No | ACME registration email. |
| `pihole_v6_tls_pem_path` | `/etc/pihole/tls.pem` | No | Combined certificate/key output path. |
| `pihole_v6_tls_cloudflare_api_token` | `""` | Yes when TLS is enabled | Cloudflare DNS API token. |
| `pihole_v6_tls_pem_owner` | `root` | No | Owner for combined PEM file. |
| `pihole_v6_tls_pem_group` | `pihole` | No | Group for combined PEM file. |
| `pihole_v6_tls_pem_mode` | `0640` | No | Mode for combined PEM file. |
| `pihole_v6_tls_cf_creds_dir` | `/root/.secrets/certbot` | No | Directory for certbot credentials file. |
| `pihole_v6_tls_cf_creds_file` | `{{ pihole_v6_tls_cf_creds_dir }}/cloudflare.ini` | No | certbot credentials file path. |
| `pihole_v6_tls_packages` | `['certbot', 'python3-certbot-dns-cloudflare']` | No | Packages required for DNS-01 cert issuance. |
| `pihole_v6_tls_acme_server` | `production` | No | ACME endpoint selector (`production` or `staging`). |
| `pihole_v6_tls_acme_directory_urls` | see defaults | No | Mapping of ACME endpoint URLs. |
| `pihole_v6_tls_dns_propagation_seconds` | `60` | No | DNS propagation wait for DNS-01. |
| `pihole_v6_tls_issue_retries` | `4` | No | Retry count for cert issuance. |
| `pihole_v6_tls_issue_delay_seconds` | `120` | No | Delay between cert issuance retries. |

## Outbound Artifacts

- `pihole.toml` at `/etc/pihole/pihole.toml`
- Optional Unbound config at `/etc/unbound/unbound.conf.d/pi-hole.conf`
- Optional TLS PEM at `{{ pihole_v6_tls_pem_path }}`
- Optional certbot deploy hook at `/etc/letsencrypt/renewal-hooks/deploy/pihole-ftl-renewal.sh`
- Pi-hole adlist state managed through `/api/lists`

## Dependencies

- Runtime role call to `ufw_profiles` when `pihole_v6_ufw_enable: true`

## Capabilities

- Installs Pi-hole in unattended mode when not already present
- Deploys managed `pihole.toml`
- Optionally installs and configures Unbound for local recursive DNS
- Optionally obtains and deploys Let's Encrypt certificates via Cloudflare DNS-01
- Manages adlists via Pi-hole v6 API and optionally triggers gravity
- Provides optional DNS and DNSSEC smoke tests

## Idempotency

idempotent: true

## Atomic

atomic: false

## Rollback

No automated rollback is implemented. Backups are enabled on template writes to allow manual rollback.

## Required Credentials

- `pihole_v6_admin_password` for API list management
- `pihole_v6_webserver_pwhash` and `pihole_v6_webserver_app_pwhash` for web/API auth configuration
- `pihole_v6_tls_cloudflare_api_token` when TLS is enabled

## Check Mode Behavior

The following tasks are skipped in check mode because they require runtime CLI execution:

- Pi-hole install script execution
- `pihole -up` update command
- `pihole -g` gravity command executions
- certbot certificate issuance
- PEM rebuild shell task

## Supported Platforms

- Ubuntu
- Debian

## Example Playbook

```yaml
---
- name: Configure Pi-hole servers
  hosts: pihole_servers
  gather_facts: false
  become: true
  roles:
    - role: khaosx.homelab.pihole_v6
      vars:
        pihole_v6_admin_password: "{{ vault_pihole_admin_password }}"
        pihole_v6_webserver_pwhash: "{{ vault_pihole_webserver_pwhash }}"
        pihole_v6_webserver_app_pwhash: "{{ vault_pihole_webserver_app_pwhash }}"
        pihole_v6_tls_cloudflare_api_token: "{{ vault_cloudflare_api_token }}"
```

## License

MIT

## Author

Kris
