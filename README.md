# Ansible Server Bootstrap

Idempotent automation for preparing an Ubuntu server for container workloads.

It configures users, SSH, UFW, Docker, NGINX, node_exporter, systemd and log rotation.

```bash
cp inventory/hosts.example.yml inventory/hosts.yml
ansible-playbook -i inventory/hosts.yml playbooks/bootstrap.yml --check
ansible-playbook -i inventory/hosts.yml playbooks/bootstrap.yml
```

Replace the example SSH key and inventory address before running.
