---
name: ansible-run-playbook
description: Procedure for checking inventory connectivity and running Ansible playbooks against webservers from the Ubuntu control node. Use when the user asks to run, test, verify, or deploy a playbook, or to check host connectivity with ansible ping.
---

# Run Ansible Playbook

Runs on the Linux control node from the repository root.

## Prerequisites

Check before running. Stop and report if any check fails.

```bash
ansible --version                       # ansible-core must be installed
test -f /etc/ansible/keys/Stockholm_3.pem && echo "key OK"
ansible-galaxy collection install -r requirements.yml
```

For WordPress, confirm that the required domain and email variables are set in `group_vars/webservers.yml`, both passwords are encrypted with Ansible Vault, and the public DNS record points to the target host. Ports 80 and 443 must be reachable for Let's Encrypt. Do not print or expose Vault values while checking configuration.

## Steps

Ask for explicit approval before each step that contacts managed hosts. A syntax check is local and does not need host approval. Review the dry-run output and obtain a separate explicit approval before applying changes.

1. Verify inventory (read-only):
   ```bash
   ansible-inventory --list
   ```
2. Check playbook syntax (local only):
   ```bash
   ansible-playbook site.yml --ask-vault-pass --syntax-check
   ```
3. Check connectivity (contacts hosts; ask for approval first):
   ```bash
   ansible webservers -m ansible.builtin.ping
   ```
4. Dry run (contacts hosts, changes nothing; ask for approval first):
   ```bash
   ansible-playbook site.yml --ask-vault-pass --check --diff
   ```
5. Real run (changes hosts; requires separate explicit approval after the dry run is reviewed):
   ```bash
   ansible-playbook site.yml --ask-vault-pass
   ```

## Verification after a real run

Verify WordPress deployment without displaying credentials:

```bash
ansible webservers -m ansible.builtin.command -a "systemctl is-active apache2 mariadb certbot.timer"
ansible webservers -m ansible.builtin.command -a "certbot certificates"
curl --fail --silent --show-error --head https://your-domain/
```

The database password is necessarily stored in the target `wp-config.php` for WordPress to connect to MariaDB; the role restricts that file to `root:www-data` with mode `0640`. WordPress stores the administrator password as a hash in the database. Never include either password in verification commands or logs.

## Failure handling

- If `ping` fails with a permission error, check the key path and `ansible_user` in `hosts`.
- If a task fails, stop and report the task name and error. Do not retry with different options without approval.
- Do not run with `--force-handlers` or `--skip-tags` unless the user asks.
- If Certbot cannot issue a certificate, stop and verify domain DNS and inbound port 80 before proposing a retry; do not bypass certificate checks.
- Check mode does not execute WP-CLI or Certbot command tasks, so successful dry-run output does not prove site installation or certificate issuance.
