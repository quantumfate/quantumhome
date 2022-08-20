# quantumhome

## Requirements

- a user configured on the target machine
- access via ssh

## Optional

- It is recommended to use [multiplexing](https://en.wikibooks.org/wiki/OpenSSH/Cookbook/Multiplexing) to speed up the processing of requests

### quick tutorial

- create the directory for controlscockets

```bash
mkdir -pv ~/.ssh/controlmasters/
```

- add the following to your `.ssh/config` file

```bash
Host *
        ControlMaster auto
        ControlPath ~/.ssh/controlmasters/%r@%h:%p
        ControlPersist yes
```

## The actual project

This playbook sets up a homeserver following the [IaC phylosophy](https://en.wikipedia.org/wiki/Infrastructure_as_code). The individual roles in this playbook are not solely meant to be run on their own but some can be. One example are [Web Container](https://github.com/quantumfate/quantumhome/tree/main/roles/web_containers) roles.

## variables

This projects makes use of lots of variable methods and concepts from ansible.
Take a look at [Ansible's "understanding variable precedence" guide.](https://docs.ansible.com/ansible/latest/user_guide/playbooks_variables.html#understanding-variable-precedence)

### group_vars and host_vars

There are 3 main places to declare variables:

- groupvars/all: secret.yml (with ansible-vault) and vars.yml
- host_vars/host: vars.yml but host specific

```bash
├── group_vars
│   └── all
├── host_vars
│   ├── quantumhome
│   └── raspberrypi
```

### container and homer specific variables

Default role varibales are defined in roles/role/defaults/main.yml
These variables can be overwritten by anything else in the playbook.

Take a look specifically at the footnotes in [Ansible's "understanding variable precedence" guide.](https://docs.ansible.com/ansible/latest/user_guide/playbooks_variables.html#understanding-variable-precedence)

```bash
├── roles
│   ├── homer
│   │   ├── defaults/main.yml
```

## Automation

There are three variables that allow you to define paths where your roles are located.

```yaml
# Path with simple roles to be executed
# Roles will be executed in path order. Roles in paths will be executed in alphabetical order.
simple_roles_paths:
  - "roles/simple_roles/system"
  - "roles/simple_roles/security"
  - "roles/simple_roles/filesystems"

# Includes a list of paths for all roles that run in a container
# Order is not so important since everything is running in a container
system_containers_roles_paths:
  - "roles/system_containers/network"

# Includes all paths for webservices to be built
webservices_paths:
  - "roles/web_containers/network"
  - "roles/web_containers/services"
```

Defining paths here will automatically include your roles into your playbook in [run.yml](run.yml). Roles in the first level of /roles directory require manual import.

```bash
├── roles
│   ├── homer
│   ├── simple_roles
│   ├── system_conainers
│   └── web_containers
```

These variables in [group_vars/all/vars.yml](https://github.com/quantumfate/quantumhome/blob/main/group_vars/all/vars.yml) allow you to disable the role import for a whole group.

```yaml
# Enable variables
## Role import Groups
enable_roles: yes
enable_systemcontainer_roles: yes
enable_webcontainer_roles: yes
```

The default value for persistent docker storage is in the following directory. It can be changed [here](https://github.com/quantumfate/quantumhome/blob/main/roles/simple_roles/system/essential_docker/defaults/main.yml)

```yaml
docker_dir: /opt/docker/data
```

### Granular control

Roles run on a host when a certain variable with the prefix "enable_" or "enable_" + the role name is set to yes on their respective host.

If the variable is not defined it will default to false and therefor the role/container wont run on the target host.

Therefor `enable_role_security: yes` in group_vars/all/vars.yml will run the role on all hosts since variables declared in group_vars/all/vars.yml are valid almost everywhere in the playbook.

Declaring `enable_role_security: yes` in host_vars/raspberrypi/vars.yml will install the security role obly on the targeted host "raspberrypi".

If a container role (e.g. System Container, Web Container) is explicitly set to "no" and that container exist on the target machine, the container will then be removed by the role[essential_docker](https://github.com/quantumfate/quantumhome/blob/main/roles/simple_roles/system/essential_docker/tasks/main.yml).

## Folderlayout

Currently all container roles in `/roles/web_container/<category>/<service>` need to have a default/main.yml file.

Here is an example for [pihole](https://github.com/quantumfate/quantumhome/tree/main/roles/web_containers/network/pihole).

```yaml
---
container_name: pihole
url: "pihole.{{host_local}}"
homer_category: "Network"
dashboard_name: "PiHole"
ip_address: "{{ '.'.join(IPv4_lan_network.split('.')[0:3]) }}.26"
```

This data will later be used in the homer role for the homer dashboard.

## Set secret variables

Create an encrypted file with ansible-vault.

```bash
cd group_cars/all # or host_vars/<your hostname>/
ansible-vault create secret.yml
ansible-vault edit secret.yml
```

```yml
host: "" # Your cloudflare zone
host_local: "" # local hostname for example .local
security_ssh_port: "" # Port used on remote machine for ssh connection
cf_email: ""
cloudflare_dns_token: ""
quantumhome_sudo_password: ""
raspberrypi_sudo_password: ""
# Nextcloud
nc_mysql_root_password: ""
nc_mysql_password: ""
# Pihole
pihole_password: ""
pihole_api_token: ""
```

## The actual playbook

The [run.yml](run.yml) file will ran a bunch of automated tasks including the sytem role. This will setup the basic system with all neccessary dependencies for docker container.

Once that is done the playbook will import a task, that includes all roles based on the paths in the "webservices_path" variable.

## Run the playbook

- to run the playboot execute:

```bash
ansible-playbook run.yml --ask-vault-pass
```

- you can run the playbook in check-mode without actually making changes

```bash
ansible-playbook run.yml --check --ask-vault-pass
```
