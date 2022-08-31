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

This playbook sets up a homeserver following the [IaC phylosophy](https://en.wikipedia.org/wiki/Infrastructure_as_code). The individual roles in this playbook are not solely meant to be run on their own but some can be.

This playbook runs all the roles against a group of hosts. The `main` role is the homeserver that hosts all the webservices and the `pi` group holds a set of raspberrypis for smaller tasks.

You need to edit the `hosts` file in the root directory of this playbook according to your infrastructure.

```text
[main]
quantumhome
[pi]
raspberrypi
```

## variables

This projects makes use of lots of variable methods and concepts from ansible.
Take a look at [Ansible's "understanding variable precedence" guide.](https://docs.ansible.com/ansible/latest/user_guide/playbooks_variables.html#understanding-variable-precedence)

- There are 3 main places to declare variables

### group_vars and host_vars

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

Default role varibales are defined in `/path/to/role/defaults/main.yml`
These variables can be overwritten if defined somewhere else in the playbook exept for roles in the same hirarchy level.

Take a look specifically at the footnotes in [Ansible's "understanding variable precedence" guide.](https://docs.ansible.com/ansible/latest/user_guide/playbooks_variables.html#understanding-variable-precedence)

```bash
├── roles
│   ├── homer
│   │   ├── defaults/main.yml
```

## Folderlayout

Currently all container roles in `/roles/webservices/<category>/<service>` need to have a default/main.yml file.

Here is an example for [pihole](https://github.com/quantumfate/quantumhome/tree/main/roles/webservices/network/pihole).

```yaml
---
container_name: pihole
url: "pihole.{{ host_local }}"
homer_category: "Network"
dashboard_name: "PiHole"
ip_address: "{{ '.'.join(IPv4_lan_network.split('.')[0:3]) }}.26"
```

This data will later be used in the homer role for the homer dashboard. The [list_services.yml](list_services.yml) task will append a dictionary from this.

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

The [run.yml](run.yml) will run the task `ssh_juggle_port` in case the security ssh port is not yet cofigured on the target hosts - that means the target hosts will only allow request from the `security_ssh_port` and deny all other requests. When the `security_ssh_port` is not configured, the playbook connects with `default_ssh_port` but sets sshd to expect incoming connections on the `security_ssh_port` in the subsequent run. Once that is done the playbook will connect to all hosts again by executing the `ssh_port_juggle`. This will make sure that the correct `security_ssh_port` is set BEFORE the iptable role will block all requests on not defined ports. This will prevent you from having to set the port manually. Allowing request from port 44 in sshd config and allowing requests from port 22 and denying requests from all other ports in iptables will lock you out of your server for ever without manually mounting the drives. As a protective measure the iptable role wont be executed if the right conditions aren't met.

The playbook will handle security and runs all roles in a specific order against the target. Consult each role Readme.md for exact explanation what the role is doing.

After the environment of the infrastructure is set up it will run my role [quantumfate.zsh](https://github.com/quantumfate/ansible-role-zsh) against the hosts to apply your desired zsh configuration. It is recommended to run this role at the end because it will install antigen bundles, hotkeys and aliases based on your installed commands.

## Run the playbook

- to run the playboot execute:

```bash
ansible-playbook run.yml --ask-vault-pass
```

- you can run the playbook in check-mode without actually making changes

```bash
ansible-playbook run.yml --check --ask-vault-pass
```
