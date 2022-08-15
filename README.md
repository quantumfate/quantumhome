# quantumhome

## Requirements

- a user configured on the target machine 
- access via ssh


## Optional

- It is recommended to use [multiplexing](https://en.wikibooks.org/wiki/OpenSSH/Cookbook/Multiplexing) to speed up the processing of requests

### brief tutorial:
- create the directory for controlscockets
```bash
mkdir -pv ~/.ssh/controlmasters/
```

- add the following to your `.ssh/config` file
```ssh
Host *
        ControlMaster auto
        ControlPath ~/.ssh/controlmasters/%r@%h:%p
        ControlPersist yes
```
# The actual project

## variables

This projects makes use of lots of variable methods and concepts from ansible.

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

main.yml is another place where only specific variables go.
They overwrite variables declared above their level because they have precedence.
```bash
├── roles
│   ├── homer
│   │   ├── defaults/main.yml
```

## Automation

There are three variables that allow you to define paths where your roles are located.

```yaml
simple_roles_paths:
  - "roles/simple_roles/system"
  - "roles/simple_roles/security"
  - "roles/simple_roles/filesystems"

# Includes a list of paths for all roles that run in a container
# Order is not so important since everything is running in a container
system_containers_roles_paths:
  - "roles/system_containers/network"

# Includes all paths for webservices to be build
web_containers_roles_paths:
  - "roles/web_containers/network"
  - "roles/web_containers/services"
```

Defining paths here will automatically include your roles into your playbook in [run.yml](run.yml).

```bash
├── roles
│   ├── homer
│   ├── simple_roles
│   ├── system_conainers
│   └── web_containers
```

### Granular control

Roles will be run on a host when a certain variable with the prefix "enable_role_" or "enable_container_" is defined and set to "yes".
If the variable is not defined it will default to false and therefor the role/container wont run on the target host.

Therefor `enable_role_security: yes` in group_vars/all/vars.yml will run the role on all hosts since variables declared in group_vars/all/vars.yml are valid almost everywhere in the playbook.

Declaring `enable_role_security: yes` in host_vars/raspberrypi/vars.yml will install the security role obly on the targeted host "raspberrypi".

## Folderlayout

Currently all container roles in `/roles/web_container/<category>/<service>` need to have a default/main.yml file. 

Here is an example for pihole under /roles/container/network/pihole.

```yaml
---
container_name: pihole

url: "pihole.{{host_local}}"

homer_category: "Network"

dashboard_name: "PiHole"

ip_adress: "{{ '.'.join(IPv4_lan_network.split('.')[0:3]) }}.26"
```

This data will later be used in the homer role for the homer dashboard.

## Set secret variables

Create an encrypted file with ansible-vault.

```
cd group_cars/all # or host_vars/<your hostname>/
ansible-vault create secret.yml
ansible-vault edit secret.yml
```

```yml
host: "" # Your cloudflare zone
host_local: "" # local hostname for example .local
cf_email: ""
cloudflare_dns_token: ""
quantumhome_sudo_password: ""
raspberrypi_sudo_password: ""
pihole_password: ""
security_ssh_port: "" # Port used on remote machine for ssh connection
```

## The actual playbook

The [run.yml](run.yml) file will ran a bunch of automated tasks including the sytem role. This will setup the basic system with all neccessary dependencies for docker container.

Once that is done the playbook will import a task, that includes all roles based on the paths in the "webservices_path" variable.

## Run the playbook

- for the first time run the command with the `-K` flag to pass the "become" password for `sudo`
```
ansible-playbook run.yml --ask-vault-pass
```

- you can run the playbook in check-mode without actually making changes
```
ansible-playbook run.yml --check --ask-vault-pass
```
