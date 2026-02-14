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

#### SQLite

To use SQLite database for a storage, add the following configuration to your `vars.yml` file (adapt to your needs):

```yaml
privatebin_config_model: SQLite
```

#### MySQL

To use MySQL for a storage, add the following configuration to your `vars.yml` file (adapt to your needs):

```yaml
privatebin_config_model: MySQL

# Set a hostname of a MySQL instance
privatebin_database_mysql_hostname: ""

# Set a username of a MySQL instance
privatebin_database_mysql_username: 'privatebin'

# Set a hostname of a MySQL instance
privatebin_database_mysql_password: 'some-password'

# Set a database name
privatebin_database_name: 'privatebin'
```

You can also configure Data Source Name (DSN) with `privatebin_config_model_database_mysql_dsn`. See [`defaults/main.yml`](../defaults/main.yml) for its default value.

#### PostgreSQL

To use PostgreSQL for a storage, add the following configuration to your `vars.yml` file (adapt to your needs):

```yaml
privatebin_config_model: PostgreSQL

# Set a hostname of a PostgreSQL instance
privatebin_database_postgres_hostname: ""

# Set a username of a PostgreSQL instance
privatebin_database_postgres_username: 'privatebin'

# Set a hostname of a PostgreSQL instance
privatebin_database_postgres_password: 'some-password'

# Set a database name
privatebin_database_name: 'privatebin'
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

### Configure the discussion feature (optional)

You can disable the discussion feature (enabled by default). To do so, add the following configuration to your `vars.yml` file:

```yaml
privatebin_config_main_discussion_enabled: false
```

**Note**: existing discussions will remain visible after disabling the feature.

### Configure the password feature (optional)

By default, users can set a password when creating a paste. To disable it, add the following configuration to your `vars.yml` file:

```yaml
privatebin_config_main_password_enabled: false
```

### Configure the file upload feature (optional)

You can enable the file upload feature (disabled by default). To do so, add the following configuration to your `vars.yml` file:

```yaml
privatebin_config_main_fileupload_enabled: true
```

### Configure the burn-after-reading feature (optional)

PrivateBin implements the burn-after-reading feature, which automatically deletes a paste as soon as it is opened. You can have the UI pre-select this feature by adding the following configuration to your `vars.yml` file:

```yaml
privatebin_config_main_burnafterreadingselected_enabled: true
```

### Configure the default template (optional)

PrivateBin includes templates (themes) for its UI. The default theme is `bootstrap5`. You can specify one by adding the following configuration to your `vars.yml` file:

```yaml
privatebin_config_main_template: bootstrap5
```

Here are the available templates ([preview](https://github.com/PrivateBin/PrivateBin/wiki/Templates#templates-included-in-privatebin)):

- bootstrap
- bootstrap-page
- bootstrap-dark
- bootstrap-dark-page
- bootstrap-compact
- bootstrap-compact-page
- bootstrap5

### Specify a URL shortener (optional)

It is possible to have the PrivateBin instance use a URL shortener such as Bit.ly and a YOURLS instance, so that users can shorten a URL of a paste with it. **It is recommended to use a self-hosted shortener only and set a password to a paste, as the shortener will leak the paste's encryption key.**

To use a public shortener, add the following configuration to your `vars.yml` file:

```yaml
# Example: https://shortener.example.com/api?link=
privatebin_config_main_urlshortener: ""
```

For URL formats, see: [https://github.com/PrivateBin/PrivateBin/wiki/Configuration#urlshortener](https://github.com/PrivateBin/PrivateBin/wiki/Configuration#urlshortener)

#### Use a private YOURLS instance with API access key

If you are using a private YOURLS instance, you might probably want to disallow a third party to use it without credentials. You can configure authentication by adding the following configuration to your `vars.yml` file:

```yaml
privatebin_config_yourlsapi_enabled: true

# Set the "signature" (access key) issued by the YOURLS instance for using the account
privatebin_config_yourlsapi_signature: ""

# Set URL of the YOURLS instance's API, called to shorten a paste URL
# Example: https://your-yourls-instance.com/yourls-api.php
privatebin_config_yourlsapi_url: ""
```

You can find the "signature" and API's URL on the "Tools" page of the YOURLS instance.

**Note**: if `privatebin_config_yourlsapi_url` is specified, `privatebin_config_main_urlshortener` will be ignored.

💡 If you are looking for an Ansible roles for YOURLS, you might be interested in [ansible-role-yourls](https://github.com/mother-of-all-self-hosting/ansible-role-yourls).

### Configure Subresource Integrity (SRI) hashes (optional)

PrivateBin implements [Subresource Integrity](https://developer.mozilla.org/en-US/docs/Web/Security/Subresource_Integrity), a security feature that enables browsers to verify that resources they fetch (for example, from a CDN) are not tampered with.

If you modified template JavaScript files, it is necessary to regenerate the hashes of them by following the method described [this section on the official wiki](https://github.com/PrivateBin/PrivateBin/wiki/Development#subresource-integrity-for-javascript-resources). After that, you can specify them with the following configuration on your `vars.yml` file:

```yaml
privatebin_config_sri: |
  js/privatebin.js = "sha512-[…]"
  js/legacy.js = "sha512-[…]"
  …
```

>[!NOTE]
> SRI disables the instance to work if it is behind a reverse proxy which alters the assets for site loading optimization, such as CloudFlare. In this case, you need to change its configuration to bypass caching. See [this FAQ entry](https://github.com/PrivateBin/PrivateBin/wiki/FAQ#how-to-make-privatebin-work-when-using-cloudflare-for-ddos-protection) for details.

### Extending the configuration

There are some additional things you may wish to configure about the service.

Take a look at:

- [`defaults/main.yml`](../defaults/main.yml) for some variables that you can customize via your `vars.yml` file

See its [configuration sample file](https://github.com/PrivateBin/PrivateBin/blob/master/cfg/conf.sample.php) and the [documentation](https://github.com/PrivateBin/PrivateBin/wiki/Configuration) for a complete list of PrivateBin's config options.

## Installing

After configuring the playbook, run the installation command of your playbook as below:

```sh
ansible-playbook -i inventory/hosts setup.yml --tags=setup-all,start
```

If you use the MASH playbook, the shortcut commands with the [`just` program](https://github.com/mother-of-all-self-hosting/mash-playbook/blob/main/docs/just.md) are also available: `just install-all` or `just setup-all`

## Usage

After running the command for installation, PrivateBin becomes available at the specified hostname.

## Troubleshooting

### Check the service's logs

You can find the logs in [systemd-journald](https://www.freedesktop.org/software/systemd/man/systemd-journald.service.html) by logging in to the server with SSH and running `journalctl -fu privatebin` (or how you/your playbook named the service, e.g. `mash-privatebin`).
