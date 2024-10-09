
# JFrog Artifactory Installation with Ansible

This guide provides instructions to install JFrog Artifactory using Ansible, including configuration details and dependencies.

Review steps in Blog [Official JFrog Ansible Collection for Artifactory & Xray](https://jfrog.com/blog/official-jfrog-ansible-collection-for-artifactory-xray/)

I did following steps from [How to install Ansible on RHEL9 Step by Step](https://medium.com/@jaine.mayank/how-to-install-ansible-on-rhel9-step-by-step-b462237f229e)

### 1. Check Python and Ansible Versions


Step-1 Install Ansible
Check the os-release of your system by running below command
```
# cat /etc/os-release
```
Open your terminal and run the below command to install Ansible.
```
# dnf install ansible-core
```
press y to continue the installation.

Once Ansible is installed check the version of Ansible by running the below command
```
# ansible --version
```
Note: This upgrades / installs `python3` as well.

- Ensure Python 3 is installed:
  ```bash
  python3 -V
  ```
---


### SSH Key Setup Between the ansible controller box  and the Artifactory / Edge Servers
2. 
- Generate SSH keys on the Ansible controller box and add them to the  nodes used for the JFrog platform:
  ```bash
  ssh-keygen
  cat ~/.ssh/id_rsa.pub >> ~/.ssh/authorized_keys
  ```
Note: I actually used the first Artifactory/Edge box i.e `sureshv-edge-1` itself as the Ansible controller box.

- Ensure target servers are listed in an `hosts.ini` file for connectivity:
  ```bash
  vi ~/hosts.ini
  ```
  Then add the following details for the 2 artifactory/ edge nodes which form the HA cluster .
```
[artifactory_servers]
artifactory-1 ansible_host=sureshv-edge-1
artifactory-2 ansible_host=sv-edge-2
```



## Install Required  Ansible Collections needed for JFrog platform

3. Run the following commands to install essential Ansible collections:
```bash
ansible-galaxy collection install community.postgresql community.general ansible.posix
ansible-galaxy collection install jfrog.platform
```


## JFrog Artifactory Role Configuration

4. Navigate to the Artifactory role’s configuration directory:
   ```bash
   cd ~/.ansible/collections/ansible_collections/jfrog/platform
   ```
5. ## Define Global Variables

- Global variables, such as database details and URLs, are stored in:
```bash
~/.ansible/collections/ansible_collections/jfrog/platform/group_vars/all/vars.yml
```
I changed:

```
## Products enabled
artifactory_enabled: true
xray_enabled: false
distribution_enabled: false
insight_enabled: false
postgres_enabled: false

# Artifactory DB details
# jdbc:postgresql://{{ hostvars[groups['postgres_servers'][0]]['ansible_host'] }}:5432/{{ artifactory_db_name }}
artifactory_db_type: postgresql
artifactory_db_driver: org.postgresql.Driver
artifactory_db_name: edge-ha
artifactory_db_user: artifactory
artifactory_db_password: password
artifactory_db_url: >-
  jdbc:postgresql://10.196.119.65:5432/{{ artifactory_db_name }}
```

6. Update `~/.ansible/collections/ansible_collections/jfrog/platform/roles/artifactory/defaults/main.yml` with license information and optional upgrade settings:
For example :
- Set `artifactory_systemyaml_override` to false if you do not want to override the system YAML.
- Enable `artifactory_upgrade_only` only if performing an upgrade.
- If SSL is enabled, set `artifactory_nginx_ssl_enabled: true` 
and   specify certificates in:
```
~/.ansible/collections/ansible_collections/jfrog/installers/roles/artifactory_nginx_ssl/defaults/main.yml
```

Note: Ensure `artifactory_nginx_enabled` is set to false since both ports 443 and 80 cannot be enabled simultaneously.


Here is my sample:

```
# The version of artifactory to install
artifactory_version: 7.90.14

# Set this to true when SSL is enabled (to use artifactory_nginx_ssl role), default to false (implies artifactory uses artifactory_nginx role )
artifactory_nginx_ssl_enabled: false

# Set this to false when ngnix is disabled, defaults to true (implies artifactory uses artifactory_nginx role )
artifactory_nginx_enabled: true

# Provide single or HA individual licenses file separated by new line and 2-space indentation and for HA, set artifactory_ha_enabled: true.
# Example: Replace <license_1> , <license_2> , <license_3> with original licenses
artifactory_licenses: |-
  bGljZW5zZTogY0hKdlpIVmpkSE02Q2lBZ1lYSjBhV1poWTNSdmNuazZDaUFnSUNCd2NtOWtkV04w
  T2lCYVdHaDNZVmhLYkdONmIyZE5ha0Y1VGxNd2QwOURNSGRQUmxGM1RVUnZkMDFFYjNkTlJtOUxZ

  bGljZW5zZTogY0hKdlpIVmpkSE02Q2lBZ1lYSjBhV1poWTNSdmNuazZDaUFnSUNCd2NtOWtkV04w
  T2lCYVdHaDNZVmhLYkdONmIyZE5ha0Y1VGxNd2QwOURNSGRQUmxGM1RVUnZkMDFFYjNkTlJtOUxZ

# To enable HA, set to true
artifactory_ha_enabled: true

# By default, all nodes are primary (CNHA) - https://www.jfrog.com/confluence/display/JFROG/High+Availability#HighAvailability-Cloud-NativeHighAvailability
artifactory_taskaffinity: any

# To enable mission-control in artifactory (>= 7.27.x) applicable only on E+ license
artifactory_mc_enabled: true

# The location where Artifactory should install
jfrog_home_directory: /opt/jfrog

# Pick the Artifactory flavour to install, can be also cpp-ce/jcr/pro
artifactory_flavour: pro

# Whether to start Artifactory
artifactory_start_service: true

artifactory_extra_java_opts: -server -Xms512m -Xmx4g -Xss256k -XX:+UseG1GC
artifactory_system_yaml_template: system.yaml.j2
artifactory_tar_file_name: jfrog-artifactory-pro-{{ artifactory_version }}-linux.tar.gz
artifactory_home: "{{ jfrog_home_directory }}/artifactory"
artifactory_tar: "https://releases.jfrog.io/artifactory/artifactory-pro/org/artifactory/pro/jfrog-artifactory-pro/\
  {{ artifactory_version }}/{{ artifactory_tar_file_name }}"
artifactory_untar_home: "{{ jfrog_home_directory }}/artifactory-{{ artifactory_flavour }}-{{ artifactory_version }}"

# Timeout in seconds for URL request
artifactory_download_timeout: 10

# Added optional variable to download from external url
postgres_driver_download: true
postgres_driver_version: 42.6.0
postgres_driver_download_url: "https://repo1.maven.org/maven2/org/postgresql/postgresql/\
  {{ postgres_driver_version }}/postgresql-{{ postgres_driver_version }}.jar"

artifactory_user: artifactory
artifactory_group: artifactory

artifactory_daemon: artifactory

artifactory_uid: 1030
artifactory_gid: 1030

# If this is an upgrade
artifactory_upgrade_only: false

# To run Artifactory with any database other than PostgreSQL, allowNonPostgresql set to true
artifactory_allowNonPostgresql: false

# Default username and password, uncomment and change to manage with ansible
# artifactory_admin_username: admin
# artifactory_admin_password: password

artifactory_service_file: /lib/systemd/system/artifactory.service

# Provide systemyaml content below with 2-space indentation
artifactory_systemyaml: |-
  configVersion: 1
  shared:
    security:
      joinKey: "{{ join_key }}"
    extraJavaOpts: "{{ artifactory_extra_java_opts }}"
    node:
      id: {{ ansible_hostname }}
      ip: {{ ansible_host }}
      taskAffinity: {{ artifactory_taskaffinity }}
      haEnabled: {{ artifactory_ha_enabled }}
    database:
      allowNonPostgresql: {{ artifactory_allowNonPostgresql }}
      type: "{{ artifactory_db_type }}"
      driver: "{{ artifactory_db_driver }}"
      url: "{{ artifactory_db_url }}"
      username: "{{ artifactory_db_user }}"
      password: "{{ artifactory_db_password }}"
  mc:
    enabled: {{ artifactory_mc_enabled }}
  router:
    entrypoints:
      internalPort: 8046

# Provide binarystore XML content below with 2-space indentation
artifactory_binarystore: |-
  {%- if artifactory_ha_enabled -%}
  <?xml version="1.0" encoding="UTF-8"?>
  <config version="2">
    <chain template="cluster-file-system"/>
  </config>
  {%- else -%}
  <?xml version="1.0" encoding="UTF-8"?>
  <config version="1">
    <chain template="file-system"/>
  </config>
  {%- endif -%}

# Note: artifactory_systemyaml_override is by default false,  if you want to change default artifactory_systemyaml
artifactory_systemyaml_override: false

# Allow artifactory user to create crontab rules
artifactory_allow_crontab: false
```


## Running the Playbook
7. 
- Verify the playbook using `--check` option :
  ```bash
  ansible-playbook -vv ~/.ansible/collections/ansible_collections/jfrog/platform/platform.yml -i hosts.ini --private-key ~/.ssh/id_rsa --check
  ```

If you get error:
```
TASK [jfrog.platform.artifactory : Configure SELinux context] ******************************************************************************************************************************************************************************
task path: /home/sureshv/.ansible/collections/ansible_collections/jfrog/platform/roles/artifactory/shared/selinux_configure_context.yml:2
An exception occurred during task execution. To see the full traceback, use -vvv. The error was: ModuleNotFoundError: No module named 'seobject'
fatal: [artifactory-1]: FAILED! => {"changed": false, "msg": "Failed to import the required Python library (policycoreutils-python) on sureshv-edge-1's Python /usr/bin/python3. Please read the module documentation and install it in the appropriate location. If the required library is installed, but Ansible is using the wrong Python interpreter, please consult the documentation on ansible_python_interpreter"}

PLAY RECAP *********************************************************************************************************************************************************************************************************************************
artifactory-1              : ok=6    changed=1    unreachable=0    failed=1    skipped=0    rescued=0    ignored=0
```
Here’s how to resolve it:

Install policycoreutils-python-utils on the target machine. On RHEL 8 and 9, this package provides the necessary SELinux tools:

```
sudo dnf install -y policycoreutils-python-utils
```

8. If you run the command again , since the --check option does not make any changes on the node you will run into below error as no nginx service is installed
```
 TASK [jfrog.platform.artifactory_nginx : Restart nginx] ************************************************************************************************************************************************************************************
task path: /home/sureshv/.ansible/collections/ansible_collections/jfrog/platform/roles/artifactory_nginx/tasks/main.yml:33
NOTIFIED HANDLER jfrog.platform.artifactory_nginx : Restart nginx for artifactory-1
META: triggered running handlers for artifactory-1

RUNNING HANDLER [jfrog.platform.artifactory_nginx : Restart nginx] *************************************************************************************************************************************************************************
task path: /home/sureshv/.ansible/collections/ansible_collections/jfrog/platform/roles/artifactory_nginx/handlers/main.yml:3
fatal: [artifactory-1]: FAILED! => {"changed": false, "msg": "Could not find the requested service nginx: host"}

PLAY RECAP *********************************************************************************************************************************************************************************************************************************
artifactory-1              : ok=16   changed=7    unreachable=0    failed=1    skipped=0    rescued=0    ignored=0

```

9. 
- Next apply the playbook without the `--check` option  to make changes:
  ```bash
  ansible-playbook -vv ~/.ansible/collections/ansible_collections/jfrog/platform/platform.yml -i hosts.ini --private-key ~/.ssh/id_rsa
  ```

10. 
- After the Artifactory / Edge HA is up  and license expired you can  update the expired license in 
`~/.ansible/collections/ansible_collections/jfrog/installers/roles/artifactory_nginx_ssl/defaults/main.yml` and rerun the playbook without the `--check` option. 

11. 
- Sometimes if the artifactory service is not fully stopped and the restart artifactory task may fail . You can fix it by  getting the pid and kill it.
```
sudo ps -ef | grep jf
```
Then issue a `kill -9 <pid>` on all the jf processes and rerun the playbook without the `--check` option. 

---
## Important Note on HA Limitations

As of now, **Xray High Availability (HA)** and **PostgreSQL HA** are not supported. If you require Xray HA, it is recommended to install it as a single node first, then manually modify the `system.yaml` file to enable HA.

---

## Following Ansible Installation (RHEL 8) did not work for me , so will remove this from this readme later.

1. Enable the repository containing Ansible 2.9 packages:
   ```bash
   sudo subscription-manager repos --enable ansible-2.9-for-rhel-8-x86_64-rpms
   sudo yum install subscription-manager
   ```
2. If the above does not work, install the EPEL repository:
   ```bash
   sudo dnf install https://dl.fedoraproject.org/pub/epel/epel-release-latest-8.noarch.rpm
   sudo yum install ansible
   ```

---


This guide should help you get JFrog Artifactory/ Edge HA  up and running smoothly.
