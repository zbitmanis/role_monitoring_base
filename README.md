# Ansible Role: etcd

## Overview
The `etcd` Ansible role automates the deployment and configuration of an etcd cluster using Podman. This role sets up systemd services, configures TLS, and defines cluster membership.

## File Structure
```
roles/
└── etcd/
    ├── tasks/
    │   └── main.yaml
    ├── defaults/
    │   └── main.yaml
    ├── handlers/
    │   └── main.yaml
    └── templates/
        └── etcd.service.j2
```

## Tasks
### main.yaml

### 1. Gather the IP address of the etcd interface
```yaml
- name: Gather the ip address of etcd interface
  ansible.builtin.set_fact:
    etcd_node_ip:  '{{ ansible_facts[etcd_node_iface].ipv4.address }}'
```
**Purpose:**
- Retrieves the IP address of the network interface specified by `etcd_node_iface`.
- Stores the IP address in the `etcd_node_ip` variable for later use.

### 2. Deploy etcd systemd unit file
```yaml
- name: Deploy etcd systemd unit file
  ansible.builtin.template:
    src: etcd.service.j2
    dest: '{{ etcd_systemd_unit_file }}'
    mode: '0644'
  notify: Restart etcd service
```
**Purpose:**
- Uses a Jinja2 template (`etcd.service.j2`) to generate the etcd systemd unit file.
- The file is saved to the path specified by `etcd_systemd_unit_file`.
- Sets permissions to `0644`.
- Triggers a handler to restart the etcd service if changes are made.

### 3. Ensure the Podman service is running
```yaml
- name: Ensure the podman service is running
  ansible.builtin.systemd:
    name: podman
    state: started
    enabled: true
```
**Purpose:**
- Starts and enables the Podman service to ensure containers can run.

## Variables

### Paths
| Variable                  | Description                          | Default Value              |
|---------------------------|--------------------------------------|----------------------------|
| `etcd_install_files_path` | Path for temporary installation files| `/var/tmp/etcd-install`     |
| `etcd_data_dir`           | etcd data directory                  | `/var/lib/etcd`            |
| `etcd_dir`                | etcd configuration directory         | `/var/etcd`                |
| `etcd_tls_dir`            | etcd TLS directory                   | `/etc/etcd/tls`            |
| `etcd_systemd_unit_file`  | Systemd unit file path for etcd      | `/etc/systemd/system/etcd.service` |

### Security and TLS
| Variable                      | Description                          | Default Value              |
|-------------------------------|--------------------------------------|----------------------------|
| `enable_tls`                  | Enables TLS for etcd communication   | `true`                     |
| `etcd_protocol`               | Protocol used (http or https)        | Dynamic based on TLS       |
| `etcd_ca_file`                | Path to CA certificate               | `${etcd_tls_dir}/etcd-ca.pem` |
| `etcd_peer_trusted_ca_file`   | Path to peer trusted CA file         | `${etcd_tls_dir}/etcd-ca.pem` |
| `etcd_cert_file`              | Path to etcd node certificate        | `${etcd_tls_dir}/etcd-node.pem` |
| `etcd_key_file`               | Path to etcd node key file           | `${etcd_tls_dir}/etcd-node.key` |
| `etcd_peer_cert_file`         | Path to etcd peer certificate        | `${etcd_tls_dir}/etcd-node.pem` |
| `etcd_peer_key_file`          | Path to etcd peer key file           | `${etcd_tls_dir}/etcd-node.key` |
| `etcd_peer_client_cert_auth`  | Enables client cert auth for peers   | `true`                     |
| `etcd_client_cert_auth`       | Enables client cert auth             | `false`                    |

### Cluster Configuration
| Variable                  | Description                          | Default Value              |
|---------------------------|--------------------------------------|----------------------------|
| `etcd_nodes`              | List of etcd nodes with IPs          | See default list below     |
| `etcd_node_idx`           | Index of current node in cluster     | `0`                        |
| `etcd_node_iface`         | Network interface for etcd           | `eth1`                     |
| `etcd_node_name`          | Current node's name                  | `${etcd_nodes[etcd_node_idx].name}` |
| `etcd_cluster`            | Cluster membership string            | Built dynamically          |

**Default etcd nodes:**
```yaml
etcd_nodes:
  - name: etcd-node-a
    ipv4: 10.0.2.160
  - name: etcd-node-b
    ipv4: 10.0.2.237
  - name: etcd-node-c
    ipv4: 10.0.2.188
```

## Handlers
- **Restart etcd service** — Restarts the etcd systemd service when notified.

## Usage
Include this role in your playbook:

```yaml
- name: Deploy etcd cluster
  hosts: etcd_nodes
  roles:
    - etcd
```

Ensure all necessary variables are defined in `group_vars` or `host_vars`.


