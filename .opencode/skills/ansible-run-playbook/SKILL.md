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
```

## Steps

Ask for approval before each step that touches hosts. Read-only checks do not need approval.

1. Verify inventory (read-only):
   ```bash
   ansible-inventory --list
   ```
2. Check connectivity (read-only, contacts hosts):
   ```bash
   ansible webservers -m ansible.builtin.ping
   ```
3. Syntax check (local only):
   ```bash
   ansible-playbook site.yml --syntax-check
   ```
4. Dry run (contacts hosts, changes nothing):
   ```bash
   ansible-playbook site.yml --check --diff
   ```
5. Real run (changes hosts, requires explicit approval after the dry run is reviewed):
   ```bash
   ansible-playbook site.yml
   ```

## Verification after a real run

Confirm the packages from `group_vars/webservers.yml` are installed:

```bash
ansible webservers -m ansible.builtin.shell -a "dpkg -l curl wget git vim htop unzip net-tools | tail -n +6"
```

## Failure handling

- If `ping` fails with a permission error, check the key path and `ansible_user` in `hosts`.
- If a task fails, stop and report the task name and error. Do not retry with different options without approval.
- Do not run with `--force-handlers` or `--skip-tags` unless the user asks.
