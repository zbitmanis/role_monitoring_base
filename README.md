# Ansible Role: monitoring_base

## Overview

The `monitoring_base` Ansible role automates the deployment and configuration of a complete monitoring stack using Podman and systemd.

This role installs and configures:

- Prometheus (metrics collection)
- Thanos (long-term storage & query layer)
- Grafana (visualization)
- Alertmanager (alert routing)
- Node Exporter (host metrics)

It supports TLS, S3-backed long-term storage, automatic Grafana provisioning, and full systemd service management.

---

## File Structure

```
roles/
└── monitoring_base/
    ├── tasks/
    │   └── main.yaml
    ├── defaults/
    │   └── main.yaml
    ├── handlers/
    │   └── main.yaml
    └── templates/
        ├── prometheus.service.j2
        ├── alertmanager.service.j2
        ├── grafana.service.j2
        ├── thanos.service.j2
        ├── thanos-store.service.j2
        ├── thanos-query.service.j2
        ├── thanos-compactor.service.j2
        ├── mon-pod.service.j2
        ├── node-exporter.service.j2
        ├── prometheus.yml.j2
        ├── alertmanager.yml.j2
        ├── grafana.ini.j2
        └── objstore.yml.j2
```

---

## Tasks

### main.yaml

### 1. Gather the IP address of monitoring interface

```yaml
- name: Gather the ip address of monitoring interface
  ansible.builtin.set_fact:
    monitoring_node_ip: "{{ ansible_facts[monitoring_node_iface].ipv4.address }}"
```

**Purpose:**

- Retrieves the IP address of the interface defined by `monitoring_node_iface`.
- Stores it in `monitoring_node_ip` for configuration templates.

---

### 2. Create Podman pod

```yaml
- name: Create podman pod for prometheus and thanos
  containers.podman.podman_pod:
    name: "{{ monitoring_pod }}"
    state: created
```

**Purpose:**

- Creates a Podman pod (`mon`) for Prometheus and Thanos components.

---

### 3. Deploy systemd unit files

Each service is deployed using Jinja2 templates and triggers handlers on change:

Example:

```yaml
- name: Deploy prometheus systemd unit file
  ansible.builtin.template:
    src: prometheus.service.j2
    dest: "{{ prometheus_systemd_unit_file }}"
    mode: "0644"
  notify:
    - Reload systemd
    - Enable monitoring services
    - Restart prometheus service
```

**Purpose:**

- Installs systemd service units.
- Reloads systemd if changed.
- Ensures services are enabled and restarted when necessary.

---

### 4. Deploy configuration files

- Prometheus configuration
- Alertmanager configuration
- Grafana configuration
- Thanos object storage configuration
- Grafana provisioning (datasources and dashboards)

Templates are rendered using default and overridden variables.

---

## Services Managed

- mon-pod
- prometheus
- alertmanager
- grafana
- thanos
- thanos-store
- thanos-query
- thanos-compactor
- node-exporter

All services:

- Are enabled on boot
- Restart automatically when configuration changes
- Run under dedicated system users

---

## Variables

### Prometheus

| Variable | Default |
|----------|---------|
| `prometheus_release` | `v3.2.0` |
| `prometheus_dir` | `/var/prometheus` |
| `prometheus_config_dir` | `/etc/prometheus` |
| `prometheus_enable_tls` | `true` |
| `prometheus_storage_block_duration` | `2h` |

---

### Grafana

| Variable | Default |
|----------|---------|
| `grafana_release` | `11.5.2` |
| `grafana_dir` | `/var/lib/grafana` |
| `grafana_config_dir` | `/etc/grafana` |
| `grafana_admin_initial_password` | `{{ vault_grafana_admin_initial_password }}` |

---

### Alertmanager

| Variable | Default |
|----------|---------|
| `alertmanager_release` | `v0.28.0` |
| `alertmanager_dir` | `/var/lib/alertmanager` |
| `monitoring_alertmanager_enabled` | `true` |

---

### Thanos

| Variable | Default |
|----------|---------|
| `thanos_release` | `v0.35.1` |
| `thanos_s3_bucket` | `rpi-thanos-metrics-s3-eu-central-1` |
| `thanos_aws_access_key` | `{{ vault_thanos_aws_access_key }}` |
| `thanos_aws_access_secret` | `{{ vault_thanos_aws_access_secret }}` |

---

### Node Exporter

| Variable | Default |
|----------|---------|
| `node_exporter_release` | `v1.9.0` |

---

## Handlers

- Reload systemd
- Restart prometheus service
- Restart alertmanager service
- Restart grafana service
- Restart thanos service
- Restart thanos store service
- Restart thanos query service
- Restart thanos compactor service
- Restart node-exporter service
- Enable monitoring services

Handlers ensure services are properly restarted and enabled after configuration changes.

---

## Usage

### 1. Install the role

Using requirements.yml:

```yaml
roles:
  - name: monitoring_base
    src: https://github.com/zbitmanis/role_monitoring_base.git
    scm: git
```

Install:

```bash
ansible-galaxy role install -r requirements.yml
```

---

### 2. Define required Vault variables

```bash
ansible-vault create group_vars/monitoring/vault.yml
```

Example:

```yaml
vault_grafana_admin_initial_password: "StrongPassword"
vault_thanos_aws_access_key: "ACCESS_KEY"
vault_thanos_aws_access_secret: "SECRET_KEY"
```

---

### 3. Example playbook

```yaml
- name: Deploy monitoring stack
  hosts: monitoring
  become: true
  roles:
    - monitoring_base
```

Run:

```bash
ansible-playbook -i inventory site.yml --ask-vault-pass
```

---

## Access

After deployment:

- Prometheus → https://<host>:9090
- Grafana → http://<host>:3000
- Thanos Query → http://<host>:19090

---

## License

MIT
