# Ansible Server Bootstrap

Idempotent automation for preparing a new Ubuntu server for container workloads. The role turns a minimal VM into a consistent application host with administrative access, baseline packages, firewall policy, Docker and NGINX.

## Managed state

- baseline packages and package metadata
- administrative user and authorized SSH keys
- OpenSSH hardening drop-in
- UFW default-deny policy and approved ports
- Docker and NGINX enabled at boot
- inventory separation from committed examples

## Prerequisites

- Ansible Core 2.15+
- Ubuntu 22.04 or 24.04 target
- initial SSH user with sudo access
- Python 3 on the target
- public SSH key for the permanent administrator

Install required collections:

```bash
ansible-galaxy collection install -r requirements.yml
```

## Inventory setup

```bash
cp inventory/hosts.example.yml inventory/hosts.yml
```

Edit the untracked inventory:

```yaml
all:
  children:
    application:
      hosts:
        app-01:
          ansible_host: 192.0.2.10
          ansible_user: ubuntu
```

Replace the placeholder key in `roles/server_bootstrap/defaults/main.yml` or override it securely through inventory or Ansible Vault.

## Connectivity and preview

```bash
ansible -i inventory/hosts.yml application -m ping
ansible-playbook -i inventory/hosts.yml playbooks/bootstrap.yml --syntax-check
ansible-playbook -i inventory/hosts.yml playbooks/bootstrap.yml --check --diff
```

Review SSH and firewall changes carefully. Ensure the new key works before removing password access.

## Apply

```bash
ansible-playbook -i inventory/hosts.yml playbooks/bootstrap.yml
```

Typical recap:

```text
app-01 : ok=8 changed=6 unreachable=0 failed=0
```

Run it a second time to confirm idempotency:

```bash
ansible-playbook -i inventory/hosts.yml playbooks/bootstrap.yml
```

Expected second recap should contain `changed=0` unless package metadata or external state changed.

## Verify the server

```bash
ssh platform@192.0.2.10
sudo sshd -t
sudo ufw status verbose
systemctl is-enabled docker nginx
systemctl is-active docker nginx
docker version
curl -I http://127.0.0.1
```

## Variable overrides

```yaml
admin_user: platform
admin_groups: [sudo, docker]
admin_authorized_keys:
  - ssh-ed25519 AAAA... administrator
ssh_port: 22
allowed_tcp_ports: [22, 80, 443, 9100]
```

Sensitive values should be stored in Ansible Vault or an external secret manager.

## Troubleshooting

- host unreachable: check inventory address, routing, SSH user and key.
- sudo failure: confirm the bootstrap account has passwordless or supplied sudo access.
- SSH handler fails: run `sshd -t` and inspect the generated drop-in.
- locked out by UFW: use provider console access and verify the SSH port rule.
- Docker group change not visible: start a new login session.
- non-idempotent task: run with `--diff` and identify the changing input.

## Recovery

Keep the provider console available during the first hardening run. Test the new administrative login in a second terminal before ending the bootstrap session.
