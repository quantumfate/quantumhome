# Tasks

Quick explanation on tasks.

## ssh_juggle_port

Trys all configured ports until a connection was successful.

## ensure_ssh_port

Sets the `ansible_ssh_port` to the `security_ssh_port` in case [ssh_jugle_port](ssh_juggle_port.yml) made changes to the `ansible_ssh_port`

## repo_arch

Sets the `repo_arch` variable based on the hosts processor architecture

## list_services

Populates the dictionaries `web_services_to_build` and `roleswithvariables_to_build`

### web_services_to_build

A list of containers hosted as a webservice used by homer and other roles

```json
container: { // container name
    role_name: "", // name of the role
    path: "", // path were role is located
    tag_primary: "", // role name
    tag_secondary: "", // parent folder
    logo: "", // displayed logo
    ip_address: "", // Accessible swag ip address
    name: "", // Homer dashboard name
    url: "", // local or public domain
    category: "" // Homer category
}
```

### roleswithvariables_to_build

A list of other roles that might have variables that need to be used somewhere else.
The initial purpose of this variable was to provide the homer variables to other tasks as the `web_services_to_build` variable provices container variables to homer. Homer just couldn't be included because it doesn't manage itself.

```json
role: { // role name
    role_name: "", // I know it is redundant, but it simplifies things
    ip_adress: "", // Accessible swag ip address
    url: "", // local or public domain
}
```
