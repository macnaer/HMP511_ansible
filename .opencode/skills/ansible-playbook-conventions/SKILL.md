---
name: ansible-playbook-conventions
description: Conventions for writing Ansible playbooks and roles in this repository. Use when creating or editing site.yml, roles, tasks, handlers, group_vars, or when the user asks to write or review an Ansible playbook.
---

# Ansible Playbook Conventions

## Module names

- Always use fully qualified collection names (FQCN): `ansible.builtin.apt`, `ansible.builtin.service`, `ansible.builtin.copy`.
- Never use short module names like `apt` or `copy`.

## Structure

```
site.yml                      # entry playbook
group_vars/<group>.yml        # variables shared by a group
roles/<role>/tasks/main.yml   # role tasks
roles/<role>/handlers/main.yml
roles/<role>/defaults/main.yml  # only if needed; prefer group_vars
```

- Put variables in `group_vars/`, not duplicated in role `defaults/`.
- Keep one source of truth for each variable.

## Idempotency

- Every task must be safe to run repeatedly.
- Use the module's `state` parameter (`present`, `absent`, `started`, `restarted`) instead of `command` or `shell` when a module exists.
- If `command` or `shell` is unavoidable, add `creates:`, `removes:`, or `changed_when:`.

## Privilege escalation

- Set `become: true` only where root is required (at play or task level).
- Do not set `become` for tasks that do not need it.

## Naming

- Task names, handler names, and comments are in English.
- Start each task name with a verb: "Install base packages", "Restart nginx".

## Verification before running

Run these in order and fix any failure before proceeding:

```bash
ansible-playbook site.yml --syntax-check
ansible-playbook site.yml --check --diff
```

Note: `--check` does not execute `command` or `shell` tasks, and does not catch errors that appear only at runtime (for example, a non-existent package name).

## Rules

- Do not run the playbook against hosts without explicit user approval.
- Do not modify `hosts` without explicit user approval.
