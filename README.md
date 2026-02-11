# ansible.role.pihole-v6

Install and configure Pi-hole v6 on Ubuntu, with optional Unbound, TLS (Let's Encrypt DNS-01 via Cloudflare), API list management, and smoke tests.

## Purpose

This role:
- Installs Pi-hole v6 non-interactively
- Optionally installs and configures Unbound (`127.0.0.1:5335`)
- Seeds `pihole.toml`
- Manages adlists via Pi-hole v6 API
- Optionally provisions TLS and deploys `/etc/pihole/tls.pem`
- Runs optional DNS smoke tests

## Requirements

- Ubuntu host
- Ansible controller with required collections/modules
- `become: true` at play/role inclusion level
- `pihole_v6_admin_password` provided via controller inventory/vault
- If `pihole_v6_enable_tls: true`, provide `pihole_v6_tls_cloudflare_api_token`
- If `pihole_v6_ufw_enable: true`, controlling playbook must include role `ufw_profiles`

## Role Execution Model

This role is intended to run subordinately from a controlling playbook.

The controlling playbook should own:
- Base host provisioning
- Shared inventory and vault structure
- Shared firewall profile catalog
- Role ordering and privilege model

## Control Playbook Contract

Role defaults are in `defaults/main.yml`.
Controller inventory/vault should provide site-specific and secret values.

### Expected upstream variables

| Variable | Required | Description |
|---|---|---|
| `site_domain` | yes | Base domain used to construct Pi-hole web/API hostnames |
| `app_admin_user_pw` | yes | Source for `pihole_v6_admin_password` default |
| `configure_ssl_email` | if TLS enabled | ACME registration email |
| `configure_ssl_cloudflare_api_token` | if TLS enabled | Cloudflare token for DNS-01 challenge |
| `vault_pihole_webserver_pwhash` | yes | Pi-hole web UI password hash |
| `vault_pihole_webserver_app_pwhash` | yes | Pi-hole app/API password hash |

### Variable naming convention

Role-owned variables use the `pihole_v6_` prefix.

## Key Variables

- `pihole_v6_install_unbound`
- `pihole_v6_enable_tls`
- `pihole_v6_enable_tests`
- `pihole_v6_ufw_enable`
- `pihole_v6_ufw_profile`
- `pihole_v6_admin_password`
- `pihole_v6_api_base_url`
- `pihole_v6_web_domain`
- `pihole_v6_adlists_present`
- `pihole_v6_trigger_gravity`
- `pihole_v6_tls_domain`
- `pihole_v6_tls_pem_path`

See `defaults/main.yml` for full defaults.

## Example Controller Play

```yaml
---
- name: Deploy Pi-hole v6
  hosts: pihole_v6_servers
  become: true
  roles:
    - role: pihole_v6
```

## License

MIT
