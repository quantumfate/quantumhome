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
## Set secret variables

Create an encrypted file with ansible.

```
cd group_cars/all # or host_vars/<your hostname>/
ansible-vault create secret.yml
ansible-vault edit secret.yml
```

```yml
host: ""
host_local: ""
cloudflare_dns_token: ""
quantumhome_sudo_password: ""
raspberrypi_sudo_password: ""
```

## Run the playbook

- for the first time run the command with the `-K` flag to pass the "become" password for `sudo`
```
ansible-playbook run.yml -K
```
- every other time
```
ansible-playbook run.yml
```