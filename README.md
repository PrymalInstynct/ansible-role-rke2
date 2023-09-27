# Ansible Role: rke2 (v3.x)

Ansible role for installing [RKE2](https://docs.rke2.io/) ("Lightweight
Kubernetes compiled for Government Use") as either a standalone server or cluster.

[![CI](https://github.com/PrymalInstynct/ansible-role-rke2/workflows/CI/badge.svg?event=push)](https://github.com/PrymalInstynct/ansible-role-rke2/actions?query=workflow%3ACI)

## Release notes

Please see [Releases](https://github.com/PrymalInstynct/ansible-role-rke2/releases)
and [CHANGELOG.md](CHANGELOG.md).

## Requirements

The host you're running Ansible from requires the following Python dependencies:

  - `python >= 3.6.0` - [See Notes below](#important-note-about-python).
  - `ansible >= 2.9.16` or `ansible-base >= 2.10.4`

You can install dependencies using the requirements.txt file in this repository:
`pip3 install -r requirements.txt`.

This role has been tested against the following Linux Distributions:

  - Alpine Linux
  - Amazon Linux 2
  - Archlinux
  - CentOS 8
  - Debian 11
  - Fedora 31
  - Fedora 32
  - Fedora 33
  - openSUSE Leap 15
  - RockyLinux 8
  - Ubuntu 20.04 LTS

## Role Variables

See "_Server (Control Plane) Configuration_" and "_Agent (Worker) Configuraion_"
below.

### Global/Cluster Variables

Below are variables that are set against all of the play hosts for environment
consistency. These are generally cluster-level configuration.

| Variable                             | Description                                                                                | Default Value                  |
|--------------------------------------|--------------------------------------------------------------------------------------------|--------------------------------|
| `rke2_state`                          | State of rke2: installed, started, stopped, downloaded, uninstalled, validated.             | installed                      |
| `rke2_release_version`                | Use a specific version of rke2, eg. `v0.2.0`. Specify `false` for stable.                   | `false`                        |
| `rke2_airgap`                         | Boolean to enable air-gapped installations                                                 | `false`                        |
| `rke2_config_file`                    | Location of the rke2 configuration file.                                                    | `/etc/rancher/rke2/config.yaml` |
| `rke2_build_cluster`                  | When multiple play hosts are available, attempt to cluster. Read notes below.              | `true`                         |
| `rke2_registration_address`           | Fixed registration address for nodes. IP or FQDN.                                          | NULL                           |
| `rke2_github_url`                     | Set the GitHub URL to install rke2 from.                                                    | https://github.com/rancher/rke2  |
| `rke2_api_url`                        | URL for K3S updates API.                                                                   | https://update.rke2.io          |
| `rke2_install_dir`                    | Installation directory for rke2.                                                            | `/usr/local/bin`               |
| `rke2_install_hard_links`             | Install using hard links rather than symbolic links.                                       | `false`                        |
| `rke2_server_config_yaml_d_files`     | A flat list of templates to supplement the `rke2_server` configuration.                     | []                             |
| `rke2_agent_config_yaml_d_files`      | A flat list of templates to supplement the `rke2_agent` configuration.                      | []                             |
| `rke2_server_manifests_urls`          | A list of URLs to deploy on the primary control plane. Read notes below.                   | []                             |
| `rke2_server_manifests_templates`     | A flat list of templates to deploy on the primary control plane.                           | []                             |
| `rke2_server_pod_manifests_urls`      | A list of URLs for installing static pod manifests on the control plane. Read notes below. | []                             |
| `rke2_server_pod_manifests_templates` | A flat list of templates for installing static pod manifests on the control plane.         | []                             |
| `rke2_use_experimental`               | Allow the use of experimental features in rke2.                                             | `false`                        |
| `rke2_use_unsupported_config`         | Allow the use of unsupported configurations in rke2.                                        | `false`                        |
| `rke2_etcd_datastore`                 | Enable etcd embedded datastore (read notes below).                                         | `false`                        |
| `rke2_debug`                          | Enable debug logging on the rke2 service.                                                   | `false`                        |
| `rke2_registries`                     | Registries configuration file content.                                                     | `{ mirrors: {}, configs:{} }`  |

### K3S Service Configuration

The below variables change how and when the systemd service unit file for K3S
is run. Use this with caution, please refer to the [systemd documentation](https://www.freedesktop.org/software/systemd/man/systemd.unit.html#%5BUnit%5D%20Section%20Options)
for more information.

| Variable               | Description                                                          | Default Value |
|------------------------|----------------------------------------------------------------------|---------------|
| `rke2_start_on_boot`    | Start rke2 on boot.                                                   | `true`        |
| `rke2_service_requires` | List of required systemd units to rke2 service unit.                  | []            |
| `rke2_service_wants`    | List of "wanted" systemd unit to rke2 (weaker than "requires").       | []\*          |
| `rke2_service_before`   | Start rke2 before a defined list of systemd units.                    | []            |
| `rke2_service_after`    | Start rke2 after a defined list of systemd units.                     | []\*          |
| `rke2_service_env_vars` | Dictionary of environment variables to use within systemd unit file. | {}            |
| `rke2_service_env_file` | Location on host of a environment file to include.                   | `false`\*\*   |

\* The systemd unit template **always** specifies `network-online.target` for
`wants` and `after`.

\*\* The file must already exist on the target host, this role will not create
nor manage the file. You can manage this file outside of the role with
pre-tasks in your Ansible playbook.

### Group/Host Variables

Below are variables that are set against individual or groups of play hosts.
Typically you'd set these at group level for the control plane or worker nodes.

| Variable           | Description                                                       | Default Value                                     |
|--------------------|-------------------------------------------------------------------|---------------------------------------------------|
| `rke2_control_node` | Specify if a host (or host group) are part of the control plane.  | `false` (role will automatically delegate a node) |
| `rke2_server`       | Server (control plane) configuration, see notes below.            | `{}`                                              |
| `rke2_agent`        | Agent (worker) configuration, see notes below.                    | `{}`                                              |

#### Server (Control Plane) Configuration

The control plane is configured with the `rke2_server` dict variable. Please
refer to the below documentation for configuration options:

https://rancher.com/docs/rke2/latest/en/installation/install-options/server-config/

The `rke2_server` dictionary variable will contain flags from the above
(removing the `--` prefix). Below is an example:

```yaml
rke2_server:
  datastore-endpoint: postgres://postgres:verybadpass@database:5432/postgres?sslmode=disable
  cluster-cidr: 172.20.0.0/16
  flannel-backend: 'none'  # This needs to be in quotes
  disable:
    - traefik
    - coredns
```

Alternatively, you can create a .yaml file and read it in to the `rke2_server`
variable as per the below example:

```yaml
rke2_server: "{{ lookup('file', 'path/to/rke2_server.yml') | from_yaml }}"
```

Check out the [Documentation](documentation/README.md) for example
configuration.

#### Agent (Worker) Configuration

Workers are configured with the `rke2_agent` dict variable. Please refer to the
below documentation for configuration options:

https://rancher.com/docs/rke2/latest/en/installation/install-options/agent-config

The `rke2_agent` dictionary variable will contain flags from the above
(removing the `--` prefix). Below is an example:

```yaml
rke2_agent:
  with-node-id: true
  node-label:
    - "foo=bar"
    - "hello=world"
```

Alternatively, you can create a .yaml file and read it in to the `rke2_agent`
variable as per the below example:

```yaml
rke2_agent: "{{ lookup('file', 'path/to/rke2_agent.yml') | from_yaml }}"
```

Check out the [Documentation](documentation/README.md) for example
configuration.

### Ansible Controller Configuration Variables

The below variables are used to change the way the role executes in Ansible,
particularly with regards to privilege escalation.

| Variable               | Description                                                    | Default Value |
|------------------------|----------------------------------------------------------------|---------------|
| `rke2_skip_validation`  | Skip all tasks that validate configuration.                    | `false`       |
| `rke2_skip_env_checks`  | Skip all tasks that check environment configuration.           | `false`       |
| `rke2_skip_post_checks` | Skip all tasks that check post execution state.                | `false`       |
| `rke2_become`           | Escalate user privileges for tasks that need root permissions. | `false`       |

#### Important note about Python

From v3 of this role, Python 3 is required on the target system as well as on
the Ansible controller. This is to ensure consistent behaviour for Ansible
tasks as Python 2 is now EOL.

If target systems have both Python 2 and Python 3 installed, it is most likely
that Python 2 will be selected by default. To ensure Python 3 is used on a
target with both versions of Python, ensure `ansible_python_interpreter` is
set in your inventory. Below is an example inventory:

```yaml
---

rke2_cluster:
  hosts:
    kube-0:
      ansible_user: ansible
      ansible_host: 10.10.9.2
      ansible_python_interpreter: /usr/bin/python3
    kube-1:
      ansible_user: ansible
      ansible_host: 10.10.9.3
      ansible_python_interpreter: /usr/bin/python3
    kube-2:
      ansible_user: ansible
      ansible_host: 10.10.9.4
      ansible_python_interpreter: /usr/bin/python3
```

#### Important note about `rke2_release_version`

If you do not set a `rke2_release_version` the latest version from the stable
channel of rke2 will be installed. If you are developing against a specific
version of rke2 you must ensure this is set in your Ansible configuration, eg:

```yaml
rke2_release_version: v1.19.3+rke21
```

It is also possible to install specific K3s "Channels", below are some
examples for `rke2_release_version`:

```yaml
rke2_release_version: false             # defaults to 'stable' channel
rke2_release_version: stable            # latest 'stable' release
rke2_release_version: testing           # latest 'testing' release
rke2_release_version: v1.19             # latest 'v1.19' release
rke2_release_version: v1.19.3+rke23      # specific release

# Specific commit
# CAUTION - only used for testing - must be 40 characters
rke2_release_version: 48ed47c4a3e420fa71c18b2ec97f13dc0659778b
```

#### Important note about `rke2_install_hard_links`

If you are using the [system-upgrade-controller](https://github.com/rancher/system-upgrade-controller)
you will need to use hard links rather than symbolic links as the controller
will not be able to follow symbolic links. This option has been added however
is not enabled by default to avoid breaking existing installations.

To enable the use of hard links, ensure `rke2_install_hard_links` is set
to `true`.

```yaml
rke2_install_hard_links: true
```

The result of this can be seen by running the following in `rke2_install_dir`:

`ls -larthi | grep -E 'rke2|ctr|ctl' | grep -vE ".sh$" | sort`

Symbolic Links:

```text
[root@node1 bin]# ls -larthi | grep -E 'rke2|ctr|ctl' | grep -vE ".sh$" | sort
3277823 -rwxr-xr-x 1 root root  52M Jul 25 12:50 rke2-v1.18.4+rke21
3279565 lrwxrwxrwx 1 root root   31 Jul 25 12:52 rke2 -> /usr/local/bin/rke2-v1.18.6+rke21
3279644 -rwxr-xr-x 1 root root  51M Jul 25 12:52 rke2-v1.18.6+rke21
3280079 lrwxrwxrwx 1 root root   31 Jul 25 12:52 ctr -> /usr/local/bin/rke2-v1.18.6+rke21
3280080 lrwxrwxrwx 1 root root   31 Jul 25 12:52 crictl -> /usr/local/bin/rke2-v1.18.6+rke21
3280081 lrwxrwxrwx 1 root root   31 Jul 25 12:52 kubectl -> /usr/local/bin/rke2-v1.18.6+rke21
```

Hard Links:

```text
[root@node1 bin]# ls -larthi | grep -E 'rke2|ctr|ctl' | grep -vE ".sh$" | sort
3277823 -rwxr-xr-x 1 root root  52M Jul 25 12:50 rke2-v1.18.4+rke21
3279644 -rwxr-xr-x 5 root root  51M Jul 25 12:52 crictl
3279644 -rwxr-xr-x 5 root root  51M Jul 25 12:52 ctr
3279644 -rwxr-xr-x 5 root root  51M Jul 25 12:52 rke2
3279644 -rwxr-xr-x 5 root root  51M Jul 25 12:52 rke2-v1.18.6+rke21
3279644 -rwxr-xr-x 5 root root  51M Jul 25 12:52 kubectl
```

#### Important note about `rke2_build_cluster`

If you set `rke2_build_cluster` to `false`, this role will install each play
host as a standalone node. An example of when you might use this would be
when building a large number of standalone IoT devices running K3s. Below is a
hypothetical situation where we are to deploy 25 Raspberry Pi devices, each a
standalone system and not a cluster of 25 nodes. To do this we'd use a playbook
similar to the below:

```yaml
- hosts: rke2_nodes  # eg. 25 RPi's defined in our inventory.
  vars:
    rke2_build_cluster: false
  roles:
     - prymalinstynct.rke2
```

#### Important note about `rke2_control_node` and High Availability (HA)

By default only one host will be defined as a control node by Ansible, If you
do not set a host as a control node, this role will automatically delegate
the first play host as a control node. This is not suitable for use within
a Production workload.

If multiple hosts have `rke2_control_node` set to `true`, you must also set
`datastore-endpoint` in `rke2_server` as the connection string to a MySQL or
PostgreSQL database, or external Etcd cluster else the play will fail.

If using TLS, the CA, Certificate and Key need to already be available on
the play hosts.

See: [High Availability with an External DB](https://rancher.com/docs/rke2/latest/en/installation/ha/)

It is also possible, though not supported, to run a single K3s control node
with a `datastore-endpoint` defined. As this is not a typically supported
configuration you will need to set `rke2_use_unsupported_config` to `true`.

Since K3s v1.19.1 it is possible to use an embedded Etcd as the backend
database, and this is done by setting `rke2_etcd_datastore` to `true`.
The best practice for Etcd is to define at least 3 members to ensure quorum is
established. In addition to this, an odd number of members is recommended to
ensure a majority in the event of a network partition. If you want to use 2
members or an even number of members, please set `rke2_use_unsupported_config`
to `true`.

#### Important note about `rke2_server_manifests_urls` and `rke2_server_pod_manifests_urls`

To deploy server manifests and server pod manifests from URL, you need to
specify a `url` and optionally a `filename` (if none provided basename is used). Below is an example of how to deploy the
Tigera operator for Calico and kube-vip.

```yaml
---

rke2_server_manifests_urls:
  - url: https://docs.projectcalico.org/archive/v3.19/manifests/tigera-operator.yaml
    filename: tigera-operator.yaml

rke2_server_pod_manifests_urls:
  - url: https://raw.githubusercontent.com/kube-vip/kube-vip/main/example/deploy/0.1.4.yaml
    filename: kube-vip.yaml

```

#### Important note about `rke2_airgap`

When deploying rke2 in an air gapped environment you should provide the `rke2` binary in `./files/`. The binary will not be downloaded from Github and will subsequently not be verified using the provided sha256 sum, nor able to verify the version that you are running. All risks and burdens associated are assumed by the user in this scenario.

## Dependencies

No dependencies on other roles.

## Example Playbooks

Example playbook, single control node running `testing` channel rke2:

```yaml
- hosts: rke2_nodes
  vars:
    rke2_release_version: testing
  roles:
     - role: prymalinstynct.rke2
```

Example playbook, Highly Available with PostgreSQL database running the latest
stable release:

```yaml
- hosts: rke2_nodes
  vars:
    rke2_registration_address: loadbalancer  # Typically a load balancer.
    rke2_server:
      datastore-endpoint: "postgres://postgres:verybadpass@database:5432/postgres?sslmode=disable"
  pre_tasks:
    - name: Set each node to be a control node
      ansible.builtin.set_fact:
        rke2_control_node: true
      when: inventory_hostname in ['node2', 'node3']
  roles:
    - role: prymalinstynct.rke2
```

## License

[BSD 3-clause](LICENSE.txt)

## Contributors

Contributions from the community are very welcome, but please read the
[contribution guidelines](CONTRIBUTING.md) before doing so, this will help
make things as streamlined as possible.

## Original Author Information

[Xan Manning](https://xan.manning.io/)
