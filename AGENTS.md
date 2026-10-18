# AGENTS.md

Project instructions for AI coding agents working in this repository.

## Project

Ansible playbooks for managing Linux hosts on AWS.

- Control node: Ubuntu Linux machine with `ansible-core` installed system-wide (tested with 2.21.x).
- Inventory: `./hosts` (group `webservers`).
- Entry playbook: `site.yml`.
- Roles: `roles/<name>/`.
- Group variables: `group_vars/<group>.yml`.
- SSH key path (`ansible_ssh_private_key_file`) is defined in `hosts` and must exist on the control node at `/etc/ansible/keys/`. The key is never committed (`/keys/*` is in `.gitignore`).

## Language

- All code comments, documentation, commit messages, and PR descriptions are written in English.
- Conversation with the user is in Ukrainian.

## Rules

1. Before any action that changes files, runs git commands, or runs `ansible-playbook` against hosts, show exactly what will be done and wait for explicit user approval.
2. Never push to `master`. Work in a feature branch and open a Pull Request.
3. Never commit secrets (SSH keys, `.pem`, passwords, tokens).
4. Do not change `hosts` or inventory variables without explicit approval.

## Commands

Run from the repository root on the control node:

```bash
ansible-playbook site.yml --syntax-check
ansible-playbook site.yml --check --diff
ansible-playbook site.yml
ansible webservers -m ping
```

Order: syntax check, then dry run, then real run (only after approval).

## Conventions

See the skills in `.opencode/skills/`:

- `ansible-playbook-conventions`: writing playbooks and roles.
- `ansible-run-playbook`: running and verifying playbooks.
- `git-pr-workflow`: branches, commits, and pull requests.
