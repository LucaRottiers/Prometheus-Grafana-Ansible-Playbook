# Ansible Playbook for Prometheus and Grafana

An Ansible playbook to install and configure both Prometheus and Grafana on a VirtualBox VM. 

## Installation

The playbook is build using the [Ansible Skeleton](https://github.com/bertvv/ansible-skeleton) by [bertvv](https://github.com/bertvv). Which provides the user with a simplified setup for an Ansible project with a development environment powered by Vagrant.

To setup the VM and run the playbook, simply navigate to the directory where the project resieds and run the command ``vagrant up`` in the terminal.

The file ``site.yml`` specifies which roles will be executed when running the playbook. By default, three roles will be installed on the server:

- Prometheus
- Pushgateway
- Grafana

## Prometheus

The first role is responisble for the installation and configuration of Prometheus.

The playbook will always install the most recent version of Prometheus unless a specific version is given in as a variable in the file ``/roles/prometheus/vars/main.yml``.

```yaml
  ---
  prometheus_version: "latest"
```

The configuration of Prometheus is based on its main config file ``/roles/prometheus/templates/prometheus.conf.j2``. This allows both Prometheus and the Pushgateway to run on the correct ports.

```yaml
  global:
    scrape_interval: 15s

  scrape_configs:
    - job_name: 'prometheus'
      static_configs:
        - targets: ['localhost:9090']

    - job_name: 'pushgateway'
      honor_labels: true
      static_configs:
        - targets: ['localhost:9091']
```

The role also sets up Prometheus as a service on the virtual machine. This allows Prometheus to automatically start up when the virtual machine is booted. A service ``prometheus.service.j2`` is created to accomodate this.

```console
  [Unit]
  Description={{ service_name }}
  Wants=network-online.target
  After=network-online.target

  [Service]
  User={{ prometheus_user }}
  Group={{ prometheus_group }}
  Restart=always
  RestartSec=2
  StartLimitInterval=0
  Type=simple
  ExecStart={{ service_exec_command }}

  [Install]
  WantedBy=multi-user.target
```

## Pushgateway

The second role is responisble for the installation and configuration of the Pushgateway. The Prometheus Pushgateway is used to push metrics directly to Prometheus rather than let Prometheus scrape the endpoint. Commonly utilized for batch jobs or to send the outptu of a script to Prometheus.

The playbook will always install the most recent version of Pushgateway unless a specific version is given in as a variable in the file ``/roles/pushgateway/vars/main.yml``.

```yaml
  ---
  pushgateway_version: "latest"
```

The configuration of the Pushgateway is included in the main configuration file of Prometheus

The role also sets up the Pushgateway as a service on the virtual machine. This allows the Pushgateway to automatically start up when the virtual machine is booted. A service ``pushgateway.service.j2`` is created to accomodate this.

```console
  [Unit]
  Description={{ service_name }}
  Wants=network-online.target
  After=network-online.target

  [Service]
  User={{ pushgateway_user }}
  Group={{ pushgateway_group }}
  Restart=always
  RestartSec=2
  StartLimitInterval=0
  Type=simple
  ExecStart={{ service_exec_command }}

  [Install]
  WantedBy=multi-user.target
```

## Grafana

The final role is responisble for the installation and configuration of Grafana.

The playbook will always install the most recent version of Grafana unless a specific version is given in as a variable in the file ``/roles/grafana/vars/main.yml``.

```yaml
  ---
  grafana_version: "latest"
```

The configuration of Grafana is based on its main config file ``/roles/grafana/templates/grafana.conf.j2``. This allows Grafana to run on the correct port and is responsible for various setting sush as authentication, TLS, altering, ...

Unlik Prometheus, Grafana is installed using an RPM, because of that there is no need to explicitly configure it as a service.
