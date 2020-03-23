# Ansible Datadog Role

The Ansible Datadog role installs and configures the Datadog Agent and integrations. Version `4` of the role installs the Datadog Agent v7 by default.

## Setup

### Requirements

Supports most Debian and RHEL-based Linux distributions, and Windows.

## Installation

Install the [Datadog role][1] from Ansible Galaxy on your Ansible server:

```shell
ansible-galaxy install datadog.datadog
```

**Note**: `ansible-galaxy` is shipped with Ansible v1.4.2+. If you're using an earlier version of Ansible, download the role directly from the project's [Github page][2].

To deploy the Datadog Agent on hosts, add to your playbook the Datadog role with your API key:

```text
- hosts: servers
  roles:
    - { role: datadog.datadog, become: yes }  # On Ansible < 1.9, use "sudo: yes" instead of "become: yes"
  vars:
    datadog_api_key: "<YOUR_DD_API_KEY>"
```

#### Role variables

| Variable                                   | Description                                                                                                                                                                                                                                                                                               |
|--------------------------------------------|-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| `datadog_api_key`                          | Your Datadog API key.                                                                                                                                                                                                                                                                                     |
| `datadog_site`                             | The site of the Datadog intake to send Agent data to. Defaults to `datadoghq.com`, set to `datadoghq.eu` to send data to the EU site. This option is only available with Agent version >= 6.6.0.                                                                                                          |
| `datadog_agent_version`                    | The pinned version of the Agent to install (optional, but recommended), for example: `7.16.0`. Setting `datadog_agent_major_version` is not needed if `datadog_agent_version` is used. **Note**: Downgrades are not supported on Windows platforms.                                                       |
| `datadog_agent_major_version`              | The major version of the Agent to install. The possible values are 5, 6, or 7 (default). If `datadog_agent_version` is set, it takes precedence otherwise the latest version of the specified major is installed. Setting `datadog_agent_major_version` is not needed if `datadog_agent_version` is used. |
| `datadog_checks`                           | YAML configuration for Agent checks to drop into: <br> - `/etc/datadog-agent/conf.d/<check_name>.d/conf.yaml` for Agent v6 and v7, <br> - `/etc/dd-agent/conf.d` for Agent v5.                                                                                                                            |
| `datadog_config`                           | Settings for the main Agent configuration file: <br> - `/etc/datadog-agent/datadog.yaml` for Agent v6 and v7,<br> - `/etc/dd-agent/datadog.conf` for Agent v5 (under the `[Main]` section).                                                                                                               |
| `datadog_config_ex`                        | (Optional) Extra INI sections to go in `/etc/dd-agent/datadog.conf` (Agent v5 only).                                                                                                                                                                                                                      |
| `datadog_apt_repo`                         | Override the default Datadog `apt` repository.                                                                                                                                                                                                                                                            |
| `datadog_apt_cache_valid_time`             | Override the default apt cache expiration time (defaults to 1 hour).                                                                                                                                                                                                                                      |
| `datadog_apt_key_url_new`                  | Override the default URL to Datadog `apt` key (key ID `382E94DE` ; the deprecated `datadog_apt_key_url` variable refers to an expired key that's been removed from the role).                                                                                                                             |
| `datadog_yum_repo`                         | Override the default Datadog `yum` repository.                                                                                                                                                                                                                                                            |
| `datadog_yum_gpgkey`                       | Override the default URL to the Datadog `yum` key used to verify Agent v5 and v6 (up to 6.13) packages (key ID `4172A230`).                                                                                                                                                                               |
| `datadog_yum_gpgkey_e09422b3`              | Override the default URL to the Datadog `yum` key used to verify Agent v6.14+ packages (key ID `E09422B3`).                                                                                                                                                                                               |
| `datadog_yum_gpgkey_e09422b3_sha256sum`    | Override the default checksum of the `datadog_yum_gpgkey_e09422b3` key.                                                                                                                                                                                                                                   |
| `datadog_zypper_repo`                      | Override the default Datadog `zypper` repository.                                                                                                                                                                                                                                                         |
| `datadog_zypper_gpgkey`                    | Override the default URL to the Datadog `zypper` key used to verify Agent v5 and v6 (up to 6.13) packages (key ID `4172A230`).                                                                                                                                                                            |
| `datadog_zypper_gpgkey_sha256sum`          | Override the default checksum of the `datadog_zypper_gpgkey` key.                                                                                                                                                                                                                                         |
| `datadog_zypper_gpgkey_e09422b3`           | Override the default URL to the Datadog `zypper` key used to verify Agent v6.14+ packages (key ID `E09422B3`).                                                                                                                                                                                            |
| `datadog_zypper_gpgkey_e09422b3_sha256sum` | Override the default checksum of the `datadog_zypper_gpgkey_e09422b3` key.                                                                                                                                                                                                                                |
| `datadog_agent_allow_downgrade`            | Set to `yes` to allow Agent downgrades on apt-based platforms (use with caution, see `defaults/main.yml` for details). **Note**: On Centos this only works with Ansible 2.4+.                                                                                                                             |
| `use_apt_backup_keyserver`                 | Set to `true` to use the backup keyserver instead of the default one.                                                                                                                                                                                                                                     |
| `datadog_enabled`                          | Set to `false` to prevent `datadog-agent` service from starting (defaults to `true`).                                                                                                                                                                                                                     |
| `datadog_additional_groups`                | Either a list, or a string containing a comma-separated list of additional groups for the `datadog_user` (Linux only).                                                                                                                                                                                    |
| `datadog_windows_ddagentuser_name`         | The name of Windows user to create/use, in the format `<domain>\<user>` (Windows only).                                                                                                                                                                                                                   |
| `datadog_windows_ddagentuser_password`     | The password used to create the user and/or register the service (Windows only).                                                                                                                                                                                                                          |

### Integrations

To configure a Datadog integration (check), add an entry to the `datadog_checks` section. The first level key is the name of the check and the value is the YAML payload to write the configuration file. Examples are provided below.

#### Process check

To define two instances for the `process` check use the configuration below. This creates the corresponding configuration files:

* Agent v6 & v7: `/etc/datadog-agent/conf.d/process.d/conf.yaml`
* Agent v5: `/etc/dd-agent/conf.d/process.yaml`

```yml
    datadog_checks:
      process:
        init_config:
        instances:
          - name: ssh
            search_string: ['ssh', 'sshd']
          - name: syslog
            search_string: ['rsyslog']
            cpu_check_interval: 0.2
            exact_match: true
            ignore_denied_access: true
```

#### Custom check

To configure a custom check use the configuration below. This creates the corresponding configuration files:

* Agent v6 & v7: `/etc/datadog-agent/conf.d/my_custom_check.d/conf.yaml`
* Agent v5: `/etc/dd-agent/conf.d/my_custom_check.yaml`

```yml
    datadog_checks:
      my_custom_check:
        init_config:
        instances:
          - some_data: true
```

#### Autodiscovery

When using Autodiscovery, there is no pre-processing nor post-processing on the YAML. This means every YAML section is added to the final configuration file, including `autodiscovery identifiers`.

The example below configures the PostgeSQL check through **Autodiscovery**:

```yml
    datadog_checks:
      postgres:
        ad_identifiers:
          - db-master
          - db-slave
        init_config:
        instances:
          - host: %%host%%
            port: %%port%%
            username: username
            password: password
```

Learn more about [Autodiscovery in the Datadog documentation][3].

## Versions

Starting with v3 of the Datadog Ansible role, when `datadog_agent_version` is used to pin a specific Agent version, the role derives per-OS version names to comply with the version naming schemes of the supported operating systems, for example `1:7.16.0-1` for Debian and SUSE based, `7.16.0-1` for Redhat-based, and `7.16.0` for Windows. This makes it possible to target hosts running different operating systems in the same Ansible run.

For instance, if you provide `datadog_agent_version: 7.16.0`, the role installs `1:7.16.0-1` on Debian and SUSE-based, `7.16.0-1` on Redhat-based, and `7.16.0` on Windows. If the version is not provided, the role uses `1` as the epoch and `1` as the release number.

Alternatively, if you provide `datadog_agent_version: 1:7.16.0-1`, the role installs `1:7.16.0-1` on Debian and SUSE-based, `7.16.0-1` on Redhat-based, and `7.16.0` on Windows.

**Agent v5 (older version)**:

The Datadog Ansible role includes support for Datadog Agent v5 for Linux only. To install Agent v5, use `datadog_agent_major_version: 5` to install the latest version of Agent v5 or set `datadog_agent_version` to an existing Agent v5.

**Note**: The `datadog_agent5` variable is obsolete and has been removed.

### Upgrade

Role upgrade from v3 to v4

The `datadog_agent_major_version` variable has been introduced, to tell the module which major version of the Agent will be installed, `7` by default.
To install Agent v5, set it to `5`. To install Agent v6, set it to `6`.

#### Linux repositories

To behavior of the `datadog_apt_repo`, `datadog_yum_repo`, and `datadog_zypper_repo` variables has been modified. When they are not set, the official Datadog repositories for the major version set in `datadog_agent_major_version` are used:

| # | Default apt repository                    | Default yum repository             | Default zypper repository               |
|---|-------------------------------------------|------------------------------------|-----------------------------------------|
| 5 | deb https://apt.datadoghq.com stable main | https://yum.datadoghq.com/rpm      | https://yum.datadoghq.com/suse/rpm      |
| 6 | deb https://apt.datadoghq.com stable 6    | https://yum.datadoghq.com/stable/6 | https://yum.datadoghq.com/suse/stable/6 |
| 7 | deb https://apt.datadoghq.com stable 7    | https://yum.datadoghq.com/stable/7 | https://yum.datadoghq.com/suse/stable/7 |

To override the default behavior, set the `datadog_apt_repo`, `datadog_yum_repo`, or `datadog_zypper_repo` variables to something else than an empty string.

If you were previously using the Agent v5 variables `datadog_agent5_apt_repo`, `datadog_agent5_yum_repo`, or `datadog_agent5_zypper_repo` to set custom Agent v5 repositories, use `datadog_apt_repo`, `datadog_yum_repo`, or `datadog_zypper_repo`(with `datadog_agent_major_version` set to `5` or `datadog_agent_version` pinned to a specific Agent v5 version) instead.

To install Agent v5 with the v4 role, follow the instructions in the [Agent v5](#agent-v5-older-version) section.
To downgrade an Agent installation with the v4 role, follow the instructions in the [Agent downgrade](#agent-version-downgrades) section.

#### Windows

To behavior of the `datadog_windows_download_url` variable has been modified. When not set, the official Windows msi package corresponding to the `datadog_agent_major_version` is used:

| # | Default Windows msi package URL                                                  |
|---|----------------------------------------------------------------------------------|
| 6 | https://s3.amazonaws.com/ddagent-windows-stable/datadog-agent-6-latest.amd64.msi |
| 7 | https://s3.amazonaws.com/ddagent-windows-stable/datadog-agent-7-latest.amd64.msi |

To override the default behavior, set the `datadog_windows_download_url` variable to something else than an empty string.

#### Integrations

**Available for Agent v6.8+**

Upgrading an integration

The `datadog_integration` resource helps you to install specific version of a Datadog integration. Keep in mind the Agent comes with all the integrations already installed. So this command is here to allow you to upgrade a specific integration without upgrading the whole Agent. For more usage information consult the [Agent documentation](https://docs.datadoghq.com/agent/guide/integration-management/).

Available actions:

* `install`: Installs a specific version of the integration.
* `remove`: Removes an integration.

Syntax:

```yml
  datadog_integration:
    <INTEGRATION_NAME>:
      action: <ACTION>
      version: <VERSION_TO_INSTALL>
```

**Example**:

This example installs version `1.11.0` of the ElasticSearch integration and removes the `postgres` integration.

```yml
 datadog_integration:
   datadog-elastic:
     action: install
     version: 1.11.0
   datadog-postgres:
     action: remove
```

In order to get the available versions of the integrations, please refer to their `CHANGELOG.md` file in the [integrations-core repository](https://github.com/DataDog/integrations-core).

### Downgrade

To downgrade to a prior version of the Agent, you need to (**on centos this will only work with ansible 2.4 and up**):

1. Set `datadog_agent_version` to a specific version to downgrade to (ex: 5.32.5),
2. Set `datadog_agent_allow_downgrade` to `yes`.

**Note:** downgrades are not supported on Windows platforms.

## Playbooks

Sending data to Datadog US (default) and configuring a few checks.

```yml
- hosts: servers
  roles:
    - { role: datadog.datadog, become: yes }
  vars:
    datadog_api_key: "123456"
    datadog_agent_version: "7.16.0"
    datadog_config:
      tags:
        - "env:dev"
        - "datacenter:local"
      log_level: INFO
      apm_config:
        enabled: true
        max_traces_per_second: 10
      logs_enabled: true  # log collection is available on Agent 6 and 7
    datadog_checks:
      process:
        init_config:
        instances:
          - name: ssh
            search_string: ['ssh', 'sshd' ]
          - name: syslog
            search_string: ['rsyslog' ]
            cpu_check_interval: 0.2
            exact_match: true
            ignore_denied_access: true
      ssh_check:
        init_config:
        instances:
          - host: localhost
            port: 22
            username: root
            password: changeme
            sftp_check: True
            private_key_file:
            add_missing_keys: True
      nginx:
        init_config:
        instances:
          - nginx_status_url: http://example.com/nginx_status/
            tags:
              - "source:nginx"
              - "instance:foo"
          - nginx_status_url: http://example2.com:1234/nginx_status/
            tags:
              - "source:nginx"
              - "instance:bar"

        #Log collection is available on Agent 6 and 7
        logs:
          - type: file
            path: /var/log/access.log
            service: myapp
            source: nginx
            sourcecategory: http_web_access
          - type: file
            path: /var/log/error.log
            service: nginx
            source: nginx
            sourcecategory: http_web_access
    # datadog_integration is available on Agent 6.8+
    datadog_integration:
      datadog-elastic:
        action: install
        version: 1.11.0
      datadog-postgres:
        action: remove
    system_probe_config:
      enabled: true
```

Example for installing the latest Agent v6:

```yml
- hosts: servers
  roles:
    - { role: datadog.datadog, become: yes }
  vars:
    datadog_agent_major_version: 6
    datadog_api_key: "123456"
```

Example for sending data to EU site:

```yml
- hosts: servers
  roles:
    - { role: datadog.datadog, become: yes }
  vars:
    datadog_site: "datadoghq.eu"
    datadog_api_key: "123456"
```

### Windows

On Windows, the `become: yes` option is not needed (and will make the role fail, as ansible won't be able to use it).

Below are two methods to make the above playbook work with Windows hosts:

### Using the inventory file (recommended)

Set the `ansible_become` option to `no` in the inventory file for each Windows host:

```ini
[servers]
linux1 ansible_host=127.0.0.1
linux2 ansible_host=127.0.0.2
windows1 ansible_host=127.0.0.3 ansible_become=no
windows2 ansible_host=127.0.0.4 ansible_become=no
```

To avoid repeating the same configuration for all Windows hosts, you can also group them and set the variable at the group level:
```ini
[linux]
linux1 ansible_host=127.0.0.1
linux2 ansible_host=127.0.0.2

[windows]
windows1 ansible_host=127.0.0.3
windows2 ansible_host=127.0.0.4

[windows:vars]
ansible_become=no
```

### Using the playbook file

Alternatively, if your playbook **only runs on Windows hosts**, you can do the following in the playbook file:

```yml
- hosts: servers
  roles:
    - { role: datadog.datadog }
  vars:
    ...
```

**Warning:** this configuration will fail on Linux hosts (as it's not setting `become: yes` for them). Only use it if the playbook is specific to Windows hosts. Otherwise use the [inventory file method](#using-the-inventory-file-recommended).

## APM

To enable APM with Agent v6 and v7 use the following configuration:

```yaml
datadog_config:
    apm_config:
        enabled: true
```

To enable APM with agent v5 use the following configuration:

```yaml
datadog_config:
    apm_enabled: "true" # has to be a string
```

## Process Agent

To control the behavior of the Process Agent, use the `enabled` variable under the `datadog_config` field. It has to be set as a string and the possible values are: `true`, `false` (for only container collection) or `disabled` (to disable the Process Agent entirely)

### Variables

The following variables are available for the Process Agent:

* `scrub_args`: Enables the scrubbing of sensitive arguments from a process command line. Default value is `true`.
* `custom_sensitive_words`: Expands the default list of sensitive words used by the cmdline scrubber.

### System Probe

The [network performance monitoring](https://docs.datadoghq.com/network_performance_monitoring/) system probe is configured under the `system_probe_config` variable.  Any variables nested underneath will be written to the `system-probe.yaml`.

Currently, the system probe only works on Linux with the Agent 6 version and beyond.

### Example of configuration
```yml
datadog_config:
  process_config:
    enabled: "true" # has to be set as a string
    scrub_args: true
    custom_sensitive_words: ['consul_token','dd_api_key']
system_probe_config:
  enabled: true
  sysprobe_socket: /opt/datadog-agent/run/sysprobe.sock
```

Once modification completed, follow the steps below:

1. Start the system-probe: `sudo service datadog-agent-sysprobe start` Note: If the service wrapper is not available on your system, run the following command instead: `sudo initctl start datadog-agent-sysprobe`

2. [Restart the Agent](https://docs.datadoghq.com/agent/guide/agent-commands/#restart-the-agent) with `sudo service datadog-agent restart`

3. Enable the system-probe to start on boot: `sudo service enable datadog-agent-sysprobe`

You may also follow the [Datadog Network Performance Monitoring documentation (NPM)](https://docs.datadoghq.com/network_performance_monitoring/installation/?tab=agent#setup) to set it up manually.

### Agent 5

To enable/disable the Process Agent on Agent 5, you need to set on `datadog_config` the `process_agent_enabled` parameter to `true`/`false`.

Set the available variables inside `process.config` under the `datadog_config_ex` field to control the Process Agent's features.

#### Example of configuration

```yml
datadog_config:
  process_agent_enabled: true
datadog_config_ex:
  process.config:
    scrub_args: true
    custom_sensitive_words: "consul_token,dd_api_key"
```

## Additional tasks

`pre_tasks` and `post_tasks` folders allow to run user defined tasks. `pre_tasks` for tasks to be executed before executing any tasks from the Datadog role and `post_tasks` for those to be executed after.

All installation tasks on all supported platforms register a `datadog_agent_install` variable that can then
be used in `post_tasks` to check the installation task's result: `datadog_agent_install.changed` is set to `true` if the installation task did install something, and `false` otherwise (for instance if the requested version was already installed).

## Troubleshooting

### dirmngr

On Debian Stretch, the `apt_key` module that the role uses requires an additional system dependency to work correctly. Unfortunately that dependency (`dirmngr`) is not provided by the module. To work around this, you can add the following configuration to the playbooks that make use of the present role:

```yml
---
- hosts: all
  pre_tasks:
    - name: Debian Stretch requires dirmngr package to be installed in order to use apt_key
      become: yes
      apt:
        name: dirmngr
        state: present

  roles:
    - { role: datadog.datadog, become: yes, datadog_api_key: "mykey" }
```

### Datadog Agent 6.14 for Windows

Due to a critical bug in Agent versions `6.14.0` and `6.14.1` on Windows, these versions have been blacklisted (starting with the version `3.3.0` of this role).

**NOTE:** Ansible will fail on Windows if `datadog_agent_version` is set to `6.14.0` or `6.14.1`. Please use `6.14.2` or above instead.

If you are updating from **6.14.0 or 6.14.1 on Windows**, we **strongly** recommend following these steps:

1. Upgrade the present `datadog.datadog` ansible role to the latest version (`>=3.3.0`)
2. Set the `datadog_agent_version` to `6.14.2` or above (by default the role install latest).

To learn more about this bug, please read [here](http://dtdg.co/win-614-fix).

[1]: https://galaxy.ansible.com/Datadog/datadog
[2]: https://github.com/DataDog/ansible-datadog
[3]: https://docs.datadoghq.com/agent/autodiscovery
