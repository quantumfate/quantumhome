# Essential Docker

This role assumes that you want to use persistant storage by default. If you don't want any of this, set the default variables to "no".

It is essential to set the value of the variable `essential_docker_backup_src` to run this role. This role expects a destination where the backupdata is pulled from.

The state of this variable doesn't matter if the varieble `host_has_backup_data` is set to "no".