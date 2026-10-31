<!--
SPDX-FileCopyrightText: 2020 Aaron Raimist
SPDX-FileCopyrightText: 2020 Chris van Dijk
SPDX-FileCopyrightText: 2020 Dominik Zajac
SPDX-FileCopyrightText: 2020 Mickaël Cornière
SPDX-FileCopyrightText: 2020-2024 MDAD project contributors
SPDX-FileCopyrightText: 2020-2024 Slavi Pantaleev
SPDX-FileCopyrightText: 2022 François Darveau
SPDX-FileCopyrightText: 2022 Julian Foad
SPDX-FileCopyrightText: 2022 Warren Bailey
SPDX-FileCopyrightText: 2023 Antonis Christofides
SPDX-FileCopyrightText: 2023 Felix Stupp
SPDX-FileCopyrightText: 2023 Julian-Samuel Gebühr
SPDX-FileCopyrightText: 2023 Pierre 'McFly' Marty
SPDX-FileCopyrightText: 2024 Thomas Miceli
SPDX-FileCopyrightText: 2024-2026 Suguru Hirahara

SPDX-License-Identifier: AGPL-3.0-or-later
-->

# Setting up PgBouncer

This is an [Ansible](https://www.ansible.com/) role which installs [PgBouncer](https://community.torproject.org/relay/setup/pgbouncer/standalone/) to run as a [Docker](https://www.docker.com/) container wrapped in a systemd service.

The standalone [Snowflake](https://pgbouncer.torproject.org/) proxy helps users connect to the [Tor](https://torproject.org/) network in places where Tor is blocked. The project claims that one of the main advantages of standalone Snowflake proxies is that they can be installed on servers and offer a higher bandwidth and more reliable option for users behind restrictive NATs and firewalls.

See the project's [documentation](https://community.torproject.org/relay/setup/pgbouncer/standalone/) to learn what PgBouncer does and why it might be useful to you.

## Adjusting the playbook configuration

To enable PgBouncer with this role, add the following configuration to your `vars.yml` file.

**Note**: the path should be something like `inventory/host_vars/mash.example.com/vars.yml` if you use the [MASH Ansible playbook](https://github.com/mother-of-all-self-hosting/mash-playbook).

```yaml
########################################################################
#                                                                      #
# pgbouncer                                                            #
#                                                                      #
########################################################################

pgbouncer_enabled: true

########################################################################
#                                                                      #
# /pgbouncer                                                           #
#                                                                      #
########################################################################
```

### Configure connections among the Postgres server, PgBouncer, and other applications

Because PgBouncer basically works as a proxy between your Postgres server and other applications, it is necessary to configure settings about connections both between the Postgres server and PgBouncer, and between Pgbouncer and other applications.

#### Configuring connections between Postgres and PgBouncer

To have PgBouncer connect to your Postgres server, add the following configuration to your `vars.yml` file:

```yaml
pgbouncer_environment_variables_databases_user: YOUR_POSTGRES_SERVER_USERNAME_HERE
pgbouncer_environment_variables_databases_password: YOUR_POSTGRES_SERVER_PASSWORD_HERE
pgbouncer_environment_variables_databases_dbname: YOUR_POSTGRES_SERVER_DATABASE_NAME_HERE
```

Make sure to replace the placeholders with your own values.

By default the role is configured to establish connection with the Postgres server via the Unix socket. You can mount the Unix socket by adding the following configuration to your `vars.yml` file:

```yaml
# Specify the path to the Postgres Unix socket path on the host (bind-mount source)
pgbouncer_database_socket_path_host: ""
```

Setting it enables to connect to the Postgres server via Unix socket mounted in the container at `/run-postgres/.s.PGSQL.5432`.

If TCP connection is preferred, connection via the Unix socket can be disabled by adding the following configuration to your `vars.yml` file:

```yaml
# Disable the connection to Postgres server via a Unix socket
pgbouncer_database_socket_enabled: false

pgbouncer_environment_variables_databases_host: YOUR_POSTGRES_SERVER_HOSTNAME_HERE
pgbouncer_environment_variables_databases_port: 5432
```

#### Configuring connections between PgBouncer and other applications

By default the role is configured to accept Unix socket connections from other applications. The Unix socket is mounted on the directory specified with `pgbouncer_run_path` on the host. You can have other applications connect to PgBouncer (and thus the Postgres server) via the mounted socket.

To disable connection via the Unix socket, add the following configuration to your `vars.yml` file:

```yaml
# Disable the connection to PgBouncer via a Unix socket
pgbouncer_container_unix_socket_enabled: false

pgbouncer_connection_hostname: YOUR_POSTGRES_SERVER_HOSTNAME_HERE
pgbouncer_connection_port: 5432
```

### Extending the configuration

There are some additional things you may wish to configure about the service.

Take a look at:

- [`defaults/main.yml`](../defaults/main.yml) for some variables that you can customize via your `vars.yml` file. You can override settings (even those that don't have dedicated playbook variables) using the `pgbouncer_environment_variables_additional_variables` variable

Refer to [this section](https://gitlab.torproject.org/tpo/anti-censorship/pluggable-transports/pgbouncer/-/tree/main/proxy?ref_type=heads#running-a-standalone-pgbouncer-proxy) for the list of options you can pass to `pgbouncer_process_extra_arguments_custom`.

## Installing

After configuring the playbook, run the installation command of your playbook as below:

```sh
ansible-playbook -i inventory/hosts setup.yml --tags=setup-all,start
```

If you use the MASH playbook, the shortcut commands with the [`just` program](https://github.com/mother-of-all-self-hosting/mash-playbook/blob/main/docs/just.md) are also available: `just install-all` or `just setup-all`

## Usage

After running the command for installation, PgBouncer becomes available.

## Troubleshooting

### Check the service's logs

You can find the logs in [systemd-journald](https://www.freedesktop.org/software/systemd/man/systemd-journald.service.html) by logging in to the server with SSH and running `journalctl -fu pgbouncer` (or how you/your playbook named the service, e.g. `mash-pgbouncer`).
