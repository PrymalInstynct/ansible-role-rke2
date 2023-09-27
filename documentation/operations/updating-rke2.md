# Updating rke2

## Before you start!

Ensure you back up your rke2 cluster. This is particularly important if you use
an external datastore or embedded Etcd. Please refer to the below guide to
backing up your rke2 datastore:

https://rancher.com/docs/rke2/latest/en/backup-restore/

Also, check your volume backups are also working!

## Proceedure

### Updates using Ansible

To update via Ansible, set `rke2_release_version` to the target version you wish
to go to. For example, from your `v1.19.3+rke21` playbook:

```yaml
---
# BEFORE

- name: Provision rke2 cluster
  hosts: rke2_cluster
  vars:
    rke2_release_version: v1.19.3+rke21
  roles:
    - name: prymalinstynct.rke2
```

Updating to `v1.20.2+rke21`:

```yaml
---
# AFTER

- name: Provision rke2 cluster
  hosts: rke2_cluster
  vars:
    rke2_release_version: v1.20.2+rke21
  roles:
    - name: prymalinstynct.rke2
```

### Automatic updates

For automatic updates, consider installing Rancher's
[system-upgrade-controller](https://rancher.com/docs/rke2/latest/en/upgrades/automated/)

**Please note**, to be able to update using the system-upgrade-controller you
will need to set `rke2_install_hard_links` to `true`.
