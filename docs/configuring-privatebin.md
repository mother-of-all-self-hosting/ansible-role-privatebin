<!--
SPDX-FileCopyrightText: 2020 - 2024 MDAD project contributors
SPDX-FileCopyrightText: 2020 - 2024 Slavi Pantaleev
SPDX-FileCopyrightText: 2020 Aaron Raimist
SPDX-FileCopyrightText: 2020 Chris van Dijk
SPDX-FileCopyrightText: 2020 Dominik Zajac
SPDX-FileCopyrightText: 2020 Mickaël Cornière
SPDX-FileCopyrightText: 2022 François Darveau
SPDX-FileCopyrightText: 2022 Julian Foad
SPDX-FileCopyrightText: 2022 Warren Bailey
SPDX-FileCopyrightText: 2023 Antonis Christofides
SPDX-FileCopyrightText: 2023 Felix Stupp
SPDX-FileCopyrightText: 2023 Pierre 'McFly' Marty
SPDX-FileCopyrightText: 2024 - 2025 Suguru Hirahara

SPDX-License-Identifier: AGPL-3.0-or-later
-->

# Setting up PrivateBin

This is an [Ansible](https://www.ansible.com/) role which installs [PrivateBin](https://privatebin.info) to run as a [Docker](https://www.docker.com/) container wrapped in a systemd service.

PrivateBin is a minimalist, open source online pastebin where the server has zero knowledge of pasted data.

See the project's [documentation](https://github.com/PrivateBin/PrivateBin/tree/master/doc) to learn what PrivateBin does and why it might be useful to you.

## Adjusting the playbook configuration

To enable PrivateBin with this role, add the following configuration to your `vars.yml` file.

**Note**: the path should be something like `inventory/host_vars/mash.example.com/vars.yml` if you use the [MASH Ansible playbook](https://github.com/mother-of-all-self-hosting/mash-playbook).

```yaml
########################################################################
#                                                                      #
# privatebin                                                           #
#                                                                      #
########################################################################

privatebin_enabled: true

########################################################################
#                                                                      #
# /privatebin                                                          #
#                                                                      #
########################################################################
```

### Set the hostname

To enable PrivateBin you need to set the hostname as well. To do so, add the following configuration to your `vars.yml` file. Make sure to replace `example.com` with your own value.

```yaml
privatebin_hostname: "example.com"
```

After adjusting the hostname, make sure to adjust your DNS records to point the domain to your server.

### Configure a storage for pastes

The role provides these storage backend options: local filesystem (default), MySQL, PostgreSQL, SQLite, Google Cloud Storage, and Amazon S3.

💡 If you are looking for Ansible roles for [MariaDB](https://mariadb.org/) and PostgreSQL, you might be interested in [ansible-role-mariadb](https://github.com/mother-of-all-self-hosting/ansible-role-mariadb) and [ansible-role-postgres](https://github.com/mother-of-all-self-hosting/ansible-role-postgres) maintained by the [Mother-of-All-Self-Hosting (MASH)](https://github.com/mother-of-all-self-hosting) team.

#### Local filesystem (default)

To use local filesystem for a storage, you need to add a Docker volume to mount in the container, so that the directory for storing files is shared with the host machine.

To add the volume, prepare a directory on the host machine and add the following configuration to your `vars.yml` file, setting the directory path to `src`:

```yaml
privatebin_container_additional_volumes:
  - type: bind
    src: /path/on/the/host
    dst: /srv/data
    options:
```

Make sure permissions of the directory specified to `src`. If not correctly specified, the service returns a permission error while trying to put data to it.

#### MySQL

To use MySQL for a storage, add the following configuration to your `vars.yml` file (adapt to your needs):

```yaml
privatebin_config_model: MySQL

# Set a hostname of a MySQL instance
privatebin_database_mysql_hostname: ''

# Set a username of a MySQL instance
privatebin_database_mysql_username: 'privatebin'

# Set a hostname of a MySQL instance
privatebin_database_mysql_password: 'some-password'

# Set a database name
privatebin_database_mysql_name: 'privatebin'
```

You can also configure Data Source Name (DSN) with `privatebin_config_model_database_mysql_dsn`. See [`defaults/main.yml`](../defaults/main.yml) for its default value.

#### PostgreSQL

To use PostgreSQL for a storage, add the following configuration to your `vars.yml` file (adapt to your needs):

```yaml
privatebin_config_model: PostgreSQL

# Set a hostname of a PostgreSQL instance
privatebin_database_postgres_hostname: ''

# Set a username of a PostgreSQL instance
privatebin_database_postgres_username: 'privatebin'

# Set a hostname of a PostgreSQL instance
privatebin_database_postgres_password: 'some-password'

# Set a database name
privatebin_database_postgres_name: 'privatebin'
```

You can also configure Data Source Name (DSN) with `privatebin_config_model_database_postgres_dsn`. See [`defaults/main.yml`](../defaults/main.yml) for its default value.

#### Google Cloud Storage

To use Google Cloud Storage, add the following configuration to your `vars.yml` file (adapt to your needs):

```yaml
privatebin_config_model: GoogleCloudStorage

# Set a Google Cloud Storage bucket
privatebin_config_model_gcs_bucket: 'my-private-bin'

# Set a Google Cloud Storage prefix
privatebin_config_model_gcs_prefix: 'pastes'
```

Before using it, authentication should be set up with [Application Default Credentials](https://cloud.google.com/docs/authentication/production#auth-cloud-implicit-nodejs).

#### Amazon S3

To use Amazon S3 object storage, add the following configuration to your `vars.yml` file (adapt to your needs):

```yaml
privatebin_config_model: S3

# Set S3 region
privatebin_config_model_s3_region: 'eu-central-1'

# Set S3 version
privatebin_config_model_s3_version: 'latest'

# Set a S3 bucket name to use
privatebin_config_model_s3_bucket: 'my-bucket'

# Set a S3 access key ID
privatebin_config_model_s3_accesskey: 'access key id'

# Set a S3 secret access key ID
privatebin_config_model_s3_secretkey: 'secret access key'
```

### Extending the configuration

There are some additional things you may wish to configure about the component.

Take a look at:

- [`defaults/main.yml`](../defaults/main.yml) for some variables that you can customize via your `vars.yml` file

For a complete list of PrivateBin's config options, see its [configuration sample file](https://github.com/PrivateBin/PrivateBin/blob/master/cfg/conf.sample.php) and the [documentation](https://github.com/PrivateBin/PrivateBin/wiki/Configuration).

## Installing

After configuring the playbook, run the installation command of your playbook as below:

```sh
ansible-playbook -i inventory/hosts setup.yml --tags=setup-all,start
```

If you use the MASH playbook, the shortcut commands with the [`just` program](https://github.com/spantaleev/matrix-docker-ansible-deploy/blob/master/docs/just.md) are also available: `just install-all` or `just setup-all`

## Usage

After running the command for installation, PrivateBin becomes available at the specified hostname.

## Troubleshooting

### Check the service's logs

You can find the logs in [systemd-journald](https://www.freedesktop.org/software/systemd/man/systemd-journald.service.html) by logging in to the server with SSH and running `journalctl -fu privatebin` (or how you/your playbook named the service, e.g. `mash-privatebin`).
