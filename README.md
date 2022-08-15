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

## Folderlayout

Currently all container roles need to have a default/main.yml file. 

Here is an example for pihole under /roles/container/network/pihole.

```yaml
---
container_name: pihole

url: "pihole.{{host_local}}"

homer_category: "Network"

dashboard_name: "PiHole"

ip_adress: "{{ '.'.join(IPv4_lan_network.split('.')[0:3]) }}.26"

Create an encrypted file with ansible.
```

## Set secret variables

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

## Run the playbook

- for the first time run the command with the `-K` flag to pass the "become" password for `sudo`
```
ansible-playbook run.yml --ask-vault-pass
```

- you can run the playbook in check-mode without actually making changes
```
ansible-playbook run.yml --check --ask-vault-pass
```
