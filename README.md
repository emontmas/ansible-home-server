# ansible-home-server

> [!IMPORTANT]
> This repository is still under early development.
> API may and will probaly evolve in future versions.
> This includes playbook and role names and variables.
> Documentation and code may be incomplete and contain TODO sections.

This repository provides a collection of playbooks and roles designed to help installing
and maintaining one or multiple linux home servers using [Ansible](www.redhat.com) automation tool.
To use this repository, you must defined what your server installation through an inventory,
then run the provided playbooks.

Playbooks and roles where designed as distro agnostic as possible by provided extended customization
through inventory variables.

For further information about Ansible concepts such as playbooks and roles, please refer
to the [Ansible documentation](https://docs.ansible.com/).

This is originally a personal project and thus primarily reflects choices made for my own home server.
Therefore some features/tools may not be (yet) supported, or some configuration may not be desired
for your use case.
You are then free to reuse part of roles or playbooks in your own playbooks!

> [!NOTE]
> Ansible controller refer to the system from which Ansible commands are run.

## Features

* Basic distribution configuration
  * Package installation
  * User creation and configuration
  * Mount points via /etc/fstab
* Firewalld configuration
* Podman container installation as system quadlet units from compose files
* Jellyfin server and QBittorrent client installation for media servers.
* NGINX reverse proxy installation and configuration via official NGINX Ansible roles.

## Server requirements

Some initial requirements are necessary on the home server to configure before using these playbooks.

* A Linux distribution must already be installed on your server. See official guides for your chosen distribution.
* The remote server must have an SSH server reachable by the Ansible contorller.
  * Ansible relies on SSH to interact with the remote servers.
* The remote server must either allow:
  * Login with the root user either with an SSH key (recommended) or a password.
  * Login with a user with sudo rights to become root, either with an SSH key (recommended) or a password.


## Set up the ansible controller

You can run playbooks either:
* directly (natively) on your distribution
* through a containerised environment with [CQFD](https://github.com/savoirfairelinux/cqfd).

These playbooks were tested with Ansible core 2.19 and 2.20.
It may work with other versions, without guarantee.

### Native

* Install Ansible via your package manager. For example, on Debian 13:

  $ apt update && apt install -y ansible

* Install the collections and roles dependencies. For example, on Debian 13:

  $ ansible-galaxy collection install -r ansible-requirements.yaml
  $ ansible-galaxy role install -r ansible-requirements.yaml

**Note:** some usual ansible collections may be install via the distribution package manager.
For example, Debian and Fedora provide the `ansible.posix`, `community.general` and `containers.podman`
collections as part of their `ansible` package.
In any case, running `ansible-galaxy collection install` won't re-install collections already
provided by a package manager.

* Install rsync. It is needed for some tasks including backup creation. For example, on Debian 13:

  $ apt update && apt install -y rsync

* **If installing podman containers from compose files:** install podlet.
It is used by the podman_quadlet_compose role.
  * See the [podlet installation instructions](https://github.com/containers/podlet#install).

### CQFD

[CQFD](https://github.com/savoirfairelinux/cqfd) is a tool that allow you to conveniently run
commands in a container from your current workdir.

To install CQFD, follow the [installation instructions](https://github.com/savoirfairelinux/cqfd#using-cqfd)
Then, initialize the provided CQFD environment by running:

  $ cqfd init

This repository provides CQFD configuration for a Debian 13 container running Ansible 2.19.

Now whenever you run an Ansible commands, just prepend it with `cqfd run`.


## Playbooks

This repository provides a set of playbooks with pre-configured server set up tasks.
You may directly reuse it, or create your own playbooks from scratch of derived from these playbooks.

### Provided playbooks

* `install-home-server-base.yaml`: set up global distro configuration.
  * Install packages.
  * Configure users.
  * Configure SSH keys for remote access.
  * Configure mount points in /etc/fstab.
  * Configure firwalld when used.
    * Add custom firewalld services.
    * Configure firewalld zones.
* `install-home-server-services.yaml`: install and configure server services.
  * Install and configure Jellyfin and QBittorrent-nox for media servers.
  * Install and configure Podman containers as quadlet units, defined as compose files by the user.
  * Install and configure NGINX web-server and reverse proxy.
  * When applicable, can restore backups created with the `backup-services.yaml` playbook.
* `backup-services.yaml`: create backup files for the services configured by `install-home-server-services.yaml`.

### How to run a playbok

To run natively, run:

  $ ansible-playbook -i <path-to-inventory> [-i <path-to-inventory>] <path-to-playbook> [<path-to-playbook>]

To use CQFD, first initialize the environment and then prepend commands with `cqfd run`:

  $ cqfd init
  $ cqfd run ansible-playbook -i <path-to-inventory> [-i <path-to-inventory>] <path-to-playbook> [<path-to-playbook>]

For more advanced behaviour, see [the ansible-playbook documentation](https://docs.ansible.com/projects/ansible/latest/cli/ansible-playbook.html)


## Inventory variables

This section document variables and groups recognised by the playbooks and roles in this repository,
as well as some usage examples.

For in-depth explanation of how ansible inventories work, please refer to
the (official Ansible documentation)[https://docs.ansible.com/projects/ansible/latest/inventory_guide/intro_inventory.html].

### Connection variables

See the Ansible documentation for more connection related variables.

```yaml
# Remote server user used by ansible to connect and perform regular tasks.
# Either root or a user with sudo rights to become root.
ansible_user: root
# Host python interpreter path
ansible_python_interpreter: /usr/bin/python3
# IP or hostname of the host used to access it when the inventory hostname doesn't.
# If not set, the inventory host name is used.
ansible_host: 192.168.1.1
```

### Groups

The following groups are used by provided playtbooks.

```yaml
# Parent group for all servers. Any host manage by this repository must be part of it.
servers:
  hosts: # ...

# Media servers with a Jellyfin server and a QBittorent client.
jellyfin_servers:
  hosts: # ...

# Servers with an NGINX server, used for example as reverse proxy.
nginx_servers:
  hosts: # ...
```

Additional groups can be created for your inventory.


### Jellyfin / QBittorent

TODO

### NGINX

TODO

### Podman quadlet pods and containers

TODO


## Roles

This repository defines some custom roles in addition from reuses community roles.
These are implicitly used in the provided playbooks, and can be reused in custom playbooks.

* `jellyfin`: installs jellyfin natively.
  * Installation from package manager or from generic release archives.
  * Backup creation and restoration.
* `qbittorrent_nox`: installs and configure QBitorrent-nox.
  * Handles basic configuration.
  * Allows user-custom configurations.
* `podman_quadlet_compose`: installs podman containers as quadlet units from a user provided compose file.
  * Easy definition of containers in a compose file.
  * Usage of the podlet utility tool to convert compose files into podman quadlet units.
    * Requires podlet on the ansible controller.
  * Rootless and rootfull containers.

For additional documentation and list of role variables, see their respective documentation.
