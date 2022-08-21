# Essential Docker

This role comes with the following default variables:

```yml
docker_persistent_data: yes # Does the host have persistent data?
docker_backup_data: no # Will the persistant data be backed up on external storage?
docker_backup_src: '' # The path where your backups are stored
docker_dir: /opt/docker/data
```

This role assumes that you want to use persistant storage by default. If you don't want any of this, set the default variables to "no". That is because most of the docker containers use persistent storage.

This role does not assume, that you want to use backups on your machine. Enable it specifically for your host.

It is essential to set the value of the variable `docker_backup_src` if you enable backups. This role expects a destination where the backupdata is pulled from. The role will fail if this requirement is not met.

## Dependencies

This role runs dependencies against your host to fully install docker based on your architecture.

- <https://github.com/geerlingguy/ansible-role-pip>
- <https://github.com/geerlingguy/ansible-role-docker>
- <https://github.com/geerlingguy/ansible-role-docker_arm> (on arm)
