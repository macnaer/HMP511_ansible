# HMP511 Ansible

Ansible playbooks for configuring Linux hosts on AWS.

## Requirements

- Control node: Ubuntu Linux with `ansible-core` installed system-wide (tested with 2.21.x).
- SSH private key `Stockholm_3.pem` at `/etc/ansible/keys/` on the control node. The key is never committed.
- Passwordless `sudo` for the `ubuntu` user on each managed host.

## Project layout

```
site.yml                      # entry playbook
hosts                         # inventory (group: webservers)
ansible.cfg                   # Ansible settings (inventory = ./hosts)
group_vars/webservers.yml     # variables for the webservers group
roles/packages/tasks/main.yml # installs base packages
AGENTS.md                     # rules for AI coding agents
.opencode/skills/             # agent skills for this repository
```

## Usage

Run from the repository root on the control node:

```bash
ansible webservers -m ansible.builtin.ping      # check connectivity
ansible-playbook site.yml --syntax-check        # syntax check
ansible-playbook site.yml --check --diff        # dry run
ansible-playbook site.yml                       # apply changes
```

Run them in this order.

## Adding packages

Edit `packages_list` in `group_vars/webservers.yml`. Do not duplicate the list in role defaults.

## Contributing

- Work in a feature branch. Never push to `master`.
- Open a Pull Request against `master`.
- Commit messages and PR descriptions are written in English.
