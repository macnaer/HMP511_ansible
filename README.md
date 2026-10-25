# HMP511 Ansible

Ansible playbooks for configuring Linux hosts on AWS.

## Requirements

- Control node: Ubuntu Linux with `ansible-core` installed system-wide (tested with 2.21.x).
- Managed WordPress host: Ubuntu Server 26.04 with inbound TCP ports 80 and 443 reachable from the internet.
- SSH private key `Stockholm_3.pem` at `/etc/ansible/keys/` on the control node. The key is never committed.
- Passwordless `sudo` for the `ubuntu` user on each managed host.
- A public domain name pointing to the managed host's public IP address before requesting a Let's Encrypt certificate.
- The `community.mysql` Ansible collection, installed from `requirements.yml` on the control node.

## Project layout

```
site.yml                      # entry playbook
hosts                         # inventory (group: webservers)
ansible.cfg                   # Ansible settings (inventory = ./hosts)
group_vars/webservers.yml     # variables for the webservers group
roles/packages/tasks/main.yml # installs base packages
roles/wordpress/              # Apache, MariaDB, PHP, WordPress and HTTPS deployment
requirements.yml              # Ansible collection dependencies
AGENTS.md                     # rules for AI coding agents
.opencode/skills/             # agent skills for this repository
```

## WordPress configuration

The `wordpress` role installs or updates Apache, MariaDB, PHP and required PHP extensions from the Ubuntu 26.04 repositories. It deploys WordPress, creates its database and administrator, configures Apache, and requests a Let's Encrypt certificate using Certbot. Certificate renewal is enabled automatically.

Set `wordpress_domain`, `wordpress_letsencrypt_email`, and `wordpress_admin_email` in `group_vars/webservers.yml`. Before certificate issuance, the domain must resolve publicly to the server and inbound TCP ports 80 and 443 must be reachable. Other non-secret role settings, such as the database name and site title, are also defined there.

Add `wordpress_db_password` and `wordpress_admin_password` to `group_vars/webservers.yml` as Ansible Vault encrypted variables. Run each command separately, enter the Vault password when prompted, provide the secret on standard input, and press Ctrl-D to finish. Paste each resulting YAML value into that file:

```bash
ansible-vault encrypt_string --ask-vault-pass --stdin-name wordpress_db_password
ansible-vault encrypt_string --ask-vault-pass --stdin-name wordpress_admin_password
```

Never put plaintext passwords or the Vault password in the repository. At playbook runtime, provide the Vault password interactively with `--ask-vault-pass` or use a securely stored Vault password file that is not committed. WP-CLI receives the initial passwords through standard input, and password-bearing Ansible tasks suppress their output. WordPress stores its administrator password as a hash in the database. The database password must also be available to WordPress at runtime, so it is written to `wp-config.php`, owned by `root:www-data` with mode `0640`.

Install the required collection on the control node:

```bash
ansible-galaxy collection install -r requirements.yml
```

Run from the repository root on the control node. After installing the collection above, run the local syntax check:

```bash
ansible-playbook site.yml --ask-vault-pass --syntax-check
```

The ping and dry-run commands contact managed hosts; get approval before running them. Run the dry run before deployment:

```bash
ansible webservers -m ansible.builtin.ping
ansible-playbook site.yml --ask-vault-pass --check --diff
```

Review the dry-run results and obtain explicit approval before applying changes:

```bash
ansible-playbook site.yml --ask-vault-pass
```

Check mode does not execute command tasks such as WP-CLI installation or certificate issuance, so it cannot verify those runtime operations.

## Adding packages

Edit `packages_list` in `group_vars/webservers.yml` to change base packages, or `wordpress_packages` to change the WordPress stack packages. Do not duplicate group variables in role defaults.

## Contributing

- Work in a feature branch. Never push to `master`.
- Open a Pull Request against `master`.
- Commit messages and PR descriptions are written in English.
