
# JFrog Artifactory Installation with Ansible

This guide provides instructions to install JFrog Artifactory using Ansible, including configuration details and dependencies.

## Prerequisites

### 1. SSH Key Setup Between Servers

- Generate SSH keys and add them to the Ansible controller server:
  ```bash
  ssh-keygen
  cat ~/.ssh/id_rsa.pub >> ~/.ssh/authorized_keys
  ```
- Ensure target servers are listed in an `hosts.ini` file for connectivity:
  ```bash
  vi hosts.ini
  ```

### 2. Check Python and Ansible Versions

- Ensure Python 3 is installed:
  ```bash
  python3 -V
  ```
- Verify if Ansible is installed; if not, proceed to step 3.

## Ansible Installation (RHEL 8)

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

## Install Required Ansible Collections

Run the following commands to install essential Ansible collections:
```bash
ansible-galaxy collection install community.postgresql community.general ansible.posix
ansible-galaxy collection install jfrog.platform
```

## JFrog Artifactory Role Configuration

1. Navigate to the Artifactory role’s configuration directory:
   ```bash
   cd ~/.ansible/collections/ansible_collections/jfrog/platform
   ```
2. Update `main.yml` with license information and optional upgrade settings:
   ```bash
   ~/.ansible/collections/ansible_collections/jfrog/platform/roles/artifactory/defaults/main.yml
   ```
   - Set `artifactory_systemyaml_override` to `false` if you do not want to override the system YAML.
   - Enable `artifactory_upgrade_only` only if performing an upgrade.

## Configure Nginx SSL Settings (if using SSL)

1. Set `artifactory_nginx_ssl_enabled: true` in `main.yml` under `roles/artifactory`.
2. If SSL is enabled, specify certificates in:
   ```bash
   ~/.ansible/collections/ansible_collections/jfrog/platform/roles/artifactory_nginx_ssl/defaults/main.yml
   ```
3. Ensure `artifactory_nginx_enabled` is set to `false` since both ports 443 and 80 cannot be enabled simultaneously.

## Define Global Variables

Global variables, such as database details and URLs, are stored in:
```bash
~/.ansible/collections/ansible_collections/jfrog/platform/group_vars/all/vars.yml
```

## Running the Playbook

- To run the playbook in check mode (without making changes):
  ```bash
  ansible-playbook -vv /root/.ansible/collections/ansible_collections/jfrog/platform/platform.yml -i hosts.ini --private-key ~/.ssh/id_rsa --check
  ```
- To apply the playbook:
  ```bash
  ansible-playbook -vv /root/.ansible/collections/ansible_collections/jfrog/platform/platform.yml -i hosts.ini --private-key ~/.ssh/id_rsa
  ```

## Important Note on HA Limitations

As of now, **Xray High Availability (HA)** and **PostgreSQL HA** are not supported. If you require Xray HA, it is recommended to install it as a single node first, then manually modify the `system.yaml` file to enable HA.

---

This guide should help you get JFrog Artifactory up and running smoothly.
