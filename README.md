# Ansible Role For Database

[![CI](https://github.com/Asya-kawai/ansible-role-database/actions/workflows/ci.yml/badge.svg)](https://github.com/Asya-kawai/ansible-role-database/actions/workflows?query=workflow%3ACI)

Setup MariaDB secure configration with specific port.

The authentication plugin is `unix_socket` by default.

# Examples

Directory tree.

```
.
├── README.md
├── ansible.cfg
├── authorized_keys
│   └── aintek
│        └── centos
│        └── ubuntu
├── group_vars
│   ├── all.yml
│   ├── webserver_centos.yml
│   └── webserver_ubuntu.yml
├── inventory
└── webservers.yml
```


## Inventory file

```
[webservers:children]
webserver_ubuntu
webserver_centos

[webserver_ubuntu]
ubuntu ansible_host=10.10.10.10 ansible_port=22

[webserver_centos]
centos ansible_host=10.10.10.11 ansible_port=22
```

## Group Vars / Common settings(all.yml)

`all.yml` sets common variables.

This role refers `mariadb-` variables.

```
# Common settings
become: yes
ansible_user: root

# Private_key is saved in local host only!
ansible_database_private_key_file: ""

mariadb_max_allowed_packet: 2GB
mariadb_connect_timeout: 10800
mariadb_net_read_timeout: 7200
mariadb_net_write_timeout: 7200
mariadb_table_definition_cache: 600
mariadb_current_root_password: 'current root password'
mariadb_new_root_password: 'new root password'
```

## Group Vars / Ubuntu(webserver_ubuntu.yml)

`webserver_ubuntu.yml` is `webservers` host's children.

If you want to add some users, use the `mariadb_users` variable.

This role refers `mariadb_users` variable having as below.

* `name` as the user name
* `password` as the user password
* `host`as the host server name
* `priv_type` as the priviledge type
* `priv_level` as the priviledge level

For `priv_type` and `priv_level`, refer to https://mariadb.com/kb/en/grant/.

```
ansible_user: ubuntu
become: yes
ansible_become_password: 'ThisIsSecret!'

mariadb_users:
  - name: application-user
    password: application-user-password
    host: localhost
    priv_type: ALL
    priv_level: '*.*'
```

## Group Vars / CentOS(webserver_centos.yml)

`webserver_centos.yml` is `webservers` host's children.

```
mariadb_users:
  - name: application-user
    password: application-user-password
    host: localhost
    priv_type: SELECT
    priv_level: '*.*'
```

# How to switch to password-based authentication from unix-socket-authentication

At the first time of switching to password-based authentication,
 You can switch between the settings as below.

```
mariadb_auth_unix_socket_plugin: false

mariadb_new_root_password: 'new root password'
```

# How to update the root password.

When updates root password, add the `mariadb_current_root_password` variable as below.

```
mariadb_auth_unix_socket_plugin: false

mariadb_new_root_password: 'new root password'
mariadb_current_root_password: 'current root password'
```

# How to DryRun and Apply

DryRun

```
ansible-playbook -i inventory --private-key="~/.database/your_private_key" -CD webservers.yml --tags database
```

Apply

```
ansible-playbook -i inventory --private-key="~/.database/your_private_key" -D webservers.yml --tags database
```
