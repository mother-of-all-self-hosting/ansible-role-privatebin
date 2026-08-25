<!--
SPDX-FileCopyrightText: 2018-2025 Slavi Pantaleev
SPDX-FileCopyrightText: 2019-2022 Aaron Raimist
SPDX-FileCopyrightText: 2019-2023 MDAD project contributors
SPDX-FileCopyrightText: 2023 QEDeD
SPDX-FileCopyrightText: 2024 Fabio Bonelli
SPDX-FileCopyrightText: 2024 Nikita Chernyi
SPDX-FileCopyrightText: 2024-2026 Suguru Hirahara
SPDX-FileCopyrightText: 2026 spatterlight

SPDX-License-Identifier: AGPL-3.0-or-later
-->

# Molecule Testing

This role supports [Molecule](https://docs.ansible.com/projects/molecule/), an Ansible testing framework designed for developing and testing Ansible collections, playbooks, and roles.

## Prerequisites

To utilize Molecule you need to prepare several requirements:

- **x86** computer running one of these operating systems that make use of [systemd](https://systemd.io/):
  - **Archlinux**
  - **CentOS**, **Rocky Linux**, **AlmaLinux**, or possibly other RHEL alternatives (although your mileage may vary)
  - **Debian** (10/Buster or newer)
  - **Ubuntu** (18.04 or newer, although [20.04 may be problematic](https://github.com/mother-of-all-self-hosting/mash-playbook/blob/main/docs/ansible.md#supported-ansible-versions) if you run the Ansible playbook on it)
- `root` access on the computer which Molecule runs against
- [Ansible](http://ansible.com/) program
- [Python](https://www.python.org/)
  - Most distributions install Python by default, but some don't (e.g. Ubuntu 18.04) and require manual installation (something like `apt-get install python3`)
- [Docker](https://www.docker.com)
  - Access to Docker UNIX socket (`/var/run/docker.sock`) is required by default

## Installation

To set up the environment for using Molecule, run the command below on the terminal:

```bash
python3 -m venv ./molecule/venv
source ./molecule/venv/bin/activate
pip3 install -r ./molecule/requirements.txt
```

## Scenarios

Currently these testing scenarios are available:

### `default`

Tests a standard PrivateBin installation.

### `default-selfbuild`

Tests a standard PrivateBin installation with self-building the container image.

Runs in CI only for branches that change a version in `defaults/main.yml`, and on demand via `workflow_dispatch`. Self-building clones and builds from source, which is slow, and it exercises nothing new until one of those versions moves.

### `mariadb`

Tests a standard PrivateBin installation with the MariaDB database.

### `postgres`

Tests a standard PrivateBin installation with the Postgres database.

## What the scenarios have to prove

The container image PrivateBin publishes ships a complete, working configuration of its own. An unconfigured `privatebin/nginx-fpm-alpine` answers `/` with 200, renders the whole application, and accepts and returns pastes over its JSON API. On top of that, `Restart=always` in the systemd unit makes `systemctl is-active` report `active` for a container that is crash-looping.

So "the unit is active" and "something answered on port 8080" say nothing about this role having done its job, and every scenario here is written to fail against that unconfigured image:

- the served page must carry the instance name the scenario configured, and the `default-src 'self'; … sandbox …` Content-Security-Policy that only this role's `privatebin_config_main_cspheader` default produces — the image emits `default-src 'none'` and no `sandbox` directive
- the version PrivateBin stamps onto the assets it serves must equal `privatebin_version`, so an image that does not actually run the pinned code is caught
- a paste posted over the JSON API must come back byte-identical, which exercises PHP, the storage backend and the configuration together
- a paste larger than `privatebin_config_main_sizelimit` must be refused — the image's own default is 10 MB and would accept it

Beyond that, each scenario proves the one thing that distinguishes it:

- `default` looks the paste up as a file below `privatebin_data_path`, which is what makes the `filesystem` model and its bind mount observable
- `mariadb` and `postgres` look the paste's row up in the database itself, as the very user and database the role handed to PrivateBin, and require `privatebin_data_path` to be completely empty — which is what rules out a silent fallback to the filesystem backend
- `default-selfbuild` reads back the image the container was created from and requires it to be the one the role built, with the upstream image absent from the host altogether

## Running

By default it is configured to run the scenarios on Ubuntu 26.04.

```bash
molecule test --scenario-name default
```

You can utilize other distributions by setting one to the `MOLECULE_DISTRO` environment variable:

```bash
# Ubuntu 24.04
MOLECULE_DISTRO=ubuntu2404 molecule test --scenario-name default

# Debian 13
MOLECULE_DISTRO=debian13 molecule test --scenario-name default

# Debian 12
MOLECULE_DISTRO=debian12 molecule test --scenario-name default
```
