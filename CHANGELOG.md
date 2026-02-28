# Changelog

## [Unreleased]

- BREAKING: Removed external inventory coupling defaults (`site_domain`, `app_admin_user_pw`, `configure_ssl_*`) from role defaults.
- Added: Full stylebook-aligned role structure (`vars/main.yml`, `tasks/set_vars.yml`, `meta/argument_specs.yml`).
- Added: Standards-compliant task and handler naming, internal variable prefixes, and check-mode gating for CLI tasks.
- Added: README contract sections (privilege, idempotency, rollback, check mode behavior, full variable spec).
- Changed: Switched service management to `ansible.builtin.systemd`.
- Changed: Replaced hardcoded reverse DNS server entries in template with `pihole_v6_dns_rev_servers` variable.
- Added: Canonical `.gitignore` for standalone graduated role repository.
