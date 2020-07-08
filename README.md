# Stone Payments - RabbitMQ

Ansible role to install and configure RabbitMQ server.

## Note

A good practice is to have, before using this, clear knowledge of what are the plugins you're going to need. By default, this role enables the management plugin, here is a example of a playbook that enables more plugins.

```yaml
---
- name: queue service
  hosts: your_hosts
  become: true
  roles:
    - role: stone-payments.rabbitmq
      rabbitmq_plugins:
        - rabbitmq_management
        - rabbitmq_shovel
        - rabbitmq_shovel_management
```

## Requirements

To install requirements you can run this command: `pipenv install --dev`

Before using this role make sure that the machines that will compose the cluster connect to each other - via ping or name resolution. Also make sure that they can connect to the Internet to download RabbitMQ package.

For RedHat machines make sure the machines are subscribed. Also, this role requires the previous installation of the EPEL repositories. Alternatively, you can use our role [stone-payments.epel](https://github.com/stone-payments/ansible-epel) to install EPEL.

## Role Variables

```yaml
  # Take the package given by the OS/distrib (Debian-like only)
  rabbitmq_os_package: false

  # RabbitMQ repositories in katello
  rabbitmq_repository_on_satellite:
    - Stone_RabbitMQ_erlang_rhel7
    - Stone_RabbitMQ_38_rhel7

  # Always install RabbitMQ,
  # unless it's already installed and you don't want it to be replaced
  rabbitmq_install_enabled: true

  # Install specific RabbitMQ version, unless it's redefined
  rabbitmq_major: 3
  rabbitmq_minor: 8
  rabbitmq_patch: 5
  rabbitmq_suffix: 1
  rabbitmq_release: "*"
  rabbitmq_version: "{{ rabbitmq_major }}.{{ rabbitmq_minor }}.{{ rabbitmq_patch }}"
  rabbitmq_package: "{{ rabbitmq_version }}-{{ rabbitmq_suffix }}"

  # TCP configuration
  rabbitmq_conf_tcp_listeners_address: ""
  rabbitmq_conf_tcp_listeners_port: 5672

  # RabbitMQ configuration
  rabbitmq_owner: rabbitmq
  rabbitmq_group: rabbitmq
  rabbitmq_home_path: /var/lib/rabbitmq/
  rabbitmq_enabled_plugins_file_path: /etc/rabbitmq/enabled_plugins
  rabbitmq_config_file_path: /etc/rabbitmq/rabbitmq.conf
  rabbitmq_env_variables_file_path: /etc/rabbitmq/rabbitmq-env.conf
  rabbitmq_conf_extra_settings:
  rabbitmq_default_loglevel: warning

  # Variables can be overriden to adapt to the user case
  rabbitmq_conf_disk_free_limit_mem_relative: 1.5
  rabbitmq_conf_vm_memory_high_watermark: 0.4
  rabbitmq_conf_num_acceptors_tcp: 10

  # System number of open files
  rabbitmq_service_d_path: /etc/systemd/system/rabbitmq-server.service.d
  rabbitmq_system_number_open_files: 50000

  # RabbitMQ cluster
  rabbitmq_erlang_cookie: ""
  rabbitmq_clustering_force: false
  rabbitmq_clustering_enabled: false
  rabbitmq_clustering_cluster_name: ""
  rabbitmq_clustering_ha_default: true
  rabbitmq_erlang_cookie_file_path: "{{ rabbitmq_home_path }}.erlang.cookie"
  rabbitmq_nodename_prefix: rabbit
  rabbitmq_nodename: "{{ ansible_fqdn }}"

  # RabbitMQ plugins
  rabbitmq_manage_plugins: true
  rabbitmq_bin_path: /usr/lib/rabbitmq/bin
  rabbitmq_sbin_path: /usr/lib/rabbitmq/sbin
  rabbitmq_plugins_prefix_path: /usr/lib/rabbitmq
  rabbitmq_plugins:
    - name: rabbitmq_management
      state: enabled
    - name: rabbitmq_shovel
      state: enabled
    - name: rabbitmq_shovel_management
      state: enabled

  # RabbitMQ Users
  rabbitmq_manage_users: true
  rabbitmq_users: []
  rabbitmq_users_default:
    - user: admin
      password: rabbitmq
      tags: administrator

  # RabbitMQ Vhosts
  rabbitmq_manage_vhosts: false
  rabbitmq_vhosts: []

  # RabbitMQ Policy
  rabbitmq_manage_policies: false
  rabbitmq_policies: []

  # Config Newrelic to monitoring RabbitMQ
  newrelic_license:
  rabbitmq_newrelic_agent_enabled: false
  rabbitmq_newrelic_agent_config_file_path: /etc/newrelic-infra/integrations.d/rabbitmq-config.yml
  rabbitmq_newrelic_command: all
  rabbitmq_newrelic_hostname: localhost
  rabbitmq_newrelic_port: 15672
  rabbitmq_newrelic_use_ssl: false
  rabbitmq_newrelic_username: admin
  rabbitmq_newrelic_password: rabbitmq
  rabbitmq_newrelic_config_path: "{{ rabbitmq_config_file_path }}"
  rabbitmq_newrelic_queues:
  rabbitmq_newrelic_exchanges:
  rabbitmq_newrelic_vhosts:
  rabbitmq_newrelic_labels:
    env: ""
    role: ""
```

## Dependencies

None yet.

## How to run this?

### Example Playbook

```yaml
  - hosts: servers
    roles:
      - stone-payments.rabbitmq
```

By default this role will install the currently last upstream version of RabbitMQ (which is 3.8.5). If you want to install any other version you must give the version numbers.

```yaml
  - hosts: servers
    roles:
      - role: stone-payments.rabbitmq
        rabbitmq_major: 3
        rabbitmq_minor: 8
        rabbitmq_patch: 5
```

By default the role will try the configure the NODENAME with `rabbit@{{ ansible_nodename }}` when clustering is enabled, if you want to customize the node name, you can substitute the NODENAME with the variables `rabbitmq_nodename_prefix`.

Others specific RabbitMQ environment variables can also be given.

```yaml
  vars:
    rabbitmq_conf_env:
      RABBITMQ_NODE_IP_ADDRESS: "127.0.0.2"
```

You can alter:

- Memory watermark (`rabbitmq_conf_disk_free_limit_mem_relative`);
- Free disk space limit (`rabbitmq_conf_vm_memory_high_watermark`);
- Number of system's open files (`rabbitmq_system_number_open_files`).

```yaml
  vars:
    rabbitmq_conf_disk_free_limit_mem_relative: 1.5
    rabbitmq_conf_vm_memory_high_watermark: 0.4
    rabbitmq_system_number_open_files: 50000
```

To create a cluster you just have to run this role against the target nodes and give some extra vars.

```yaml
- name: queue service clustered
  hosts: group2
  become: true
  roles:
    - role: stone-payments.rabbitmq
      rabbitmq_erlang_cookie: <your_cookie>
      rabbitmq_clustering_enabled: true
```

To create a cluster using FQDN for hosts, just set USE_LONGNAME.

```yaml
  vars:
    rabbitmq_conf_env:
      USE_LONGNAME: "true"
```

Notice that the cookie is hash string that can be of any size. A good practice is use a hash of 20 characters. This is the syncronization cookie used by erlang to create the cluster.

Use this [playbook](playbook.yml) as a practical example.

## Testing

This role implements unit tests with [`molecule`](https://molecule.readthedocs.io/en/latest/) and [`testinfra`](https://testinfra.readthedocs.io/en/latest/). Notice that we only support Molecule 2.0 or greater. You can install molecule with:

```shell
pipenv install --dev --three
```

After having Molecule setup, you can run the tests with this steps:

```sh
molecule test [-s scenario_name]
```

## To Do

- Add tests for the conection/read/write of rabbit's queues
- Test removing flush_handlers from clustering step
- Bug: when the master is down and the clustering step is run all the other nodes will go down.

## Contributing

Just open a PR. We love PRs!

## License

MIT
