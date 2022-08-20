# Essential Docker
This role comes with the following default variables:
```yml
host_has_persistent_data: yes
host_has_backup_data: no
essential_docker_backup_src: '' # The path where your backups are stored
docker_dir: /opt/docker/data
```
This role assumes that you want to use persistant storage by default. If you don't want any of this, set the default variables to "no".

This role does not assume, that you want to use backups on your machine. Enable it specifically for your host, or for all hosts.

It is essential to set the value of the variable `essential_docker_backup_src` if you enable backups. This role expects a destination where the backupdata is pulled from.

## Dependencies

This role runs dependencies against your host to completely install docker based on you architecture.

 - https://github.com/geerlingguy/ansible-role-pip
 - https://github.com/geerlingguy/ansible-role-docker
 - https://github.com/geerlingguy/ansible-role-docker_arm (on arm)