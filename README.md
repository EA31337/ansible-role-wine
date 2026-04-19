# Ansible Role: Wine

[![Check](https://img.shields.io/github/actions/workflow/status/EA31337/ansible-role-wine/check.yml?label=Check)](https://github.com/EA31337/ansible-role-wine/actions/workflows/check.yml)
[![Dev](https://img.shields.io/github/actions/workflow/status/EA31337/ansible-role-wine/devcontainer-ci.yml?label=Dev)](https://github.com/EA31337/ansible-role-wine/actions/workflows/devcontainer-ci.yml)
[![Molecule](https://img.shields.io/github/actions/workflow/status/EA31337/ansible-role-wine/molecule.yml?label=Molecule)](https://github.com/EA31337/ansible-role-wine/actions/workflows/molecule.yml)
[![Test](https://img.shields.io/github/actions/workflow/status/EA31337/ansible-role-wine/test.yml?label=Test)](https://github.com/EA31337/ansible-role-wine/actions/workflows/test.yml)
[![Pull Requests](https://img.shields.io/github/issues-pr/EA31337/ansible-role-wine.svg)](https://github.com/EA31337/ansible-role-wine/pulls)
[![License](https://img.shields.io/badge/license-GPLv3-brightgreen.svg)](LICENSE)

Ansible role to install Wine on UNIX-like platforms.
Wine (originally an acronym for "Wine Is Not an Emulator")
is a compatibility layer capable of running Windows applications
on several UNIX-like operating systems.

## Requirements

This role requires:

- Ansible
- Python
- Administrative/root access on target hosts
- One of the following operating systems:
  - Alpine Linux
  - Debian/Ubuntu
  - NixOS or systems with Nix package manager

## Install

### Install from Ansible Galaxy

To install this role from Ansible Galaxy, run the following command:

```console
ansible-galaxy install ea31337.wine
```

### Install from GitHub

To install this role from GitHub, run:

```console
ansible-galaxy install git+https://github.com/EA31337/ansible-role-wine.git
```

## Role Variables

Available variables are listed below, along with default values (see [defaults/main.yml](defaults/main.yml)):

```yaml
# Wine package release to install (stable, devel, or staging)
wine_release: stable

# Override Ubuntu/Debian release codename for Wine repository
# Use this to install older Wine versions from previous release repos
# Example: 'jammy' for Ubuntu 22.04 to get Wine 10.x on newer Ubuntu
wine_release_codename: ""

# Specific Wine package version to install (optional)
# For Ubuntu/Debian: can be either simple (10.0) or full (10.0.0.0~jammy-1)
# For Alpine: package version format (e.g., 10.0)
# For NixOS: version suffix (e.g., 10.0)
wine_version: ""

# Whether to install recommended packages
wine_install_recommends: false

# Whether to install winetricks
wine_install_winetricks: false
```

### Example: Installing Wine 10.0 on Ubuntu 24.04

To install Wine 10.0 on Ubuntu Noble (24.04) using the jammy repository:

```yaml
- hosts: all
  roles:
    - role: ea31337.wine
      vars:
        wine_release: stable
        wine_release_codename: jammy
        wine_version: "10.0"  # Automatically expands to 10.0.0.0~jammy-1
```

Or if you want to specify the exact version:

```yaml
- hosts: all
  roles:
    - role: ea31337.wine
      vars:
        wine_release: stable
        wine_release_codename: jammy
        wine_version: "10.0.0.0~jammy-1"  # Full version string
```

## Testing

### Docker

Steps to test role on Docker containers.

1. Install the current role by running the following commands in shell:

    ```shell
    ansible-galaxy install -r requirements.yml
    jinja2 requirements-local.yml.j2 -D "pwd=$PWD" -o requirements-local.yml
    ansible-galaxy install -r requirements-local.yml
    ```

    Alternatively, for development purposes, you can use a symbolic link, e.g.

    ```shell
    mkdir -p ~/.ansible/roles
    ln -vs "$PWD" ~/.ansible/roles/ea31337.wine
    ```

2. Ensure Docker service (e.g. Docker Desktop) is running.
3. Run playbook from `tests/`:

    ```shell
    ansible-playbook -i tests/inventory/docker-containers.yml tests/playbooks/docker-containers.yml
    ```

### Molecule

To test using Molecule, run:

```shell
molecule test
```

## License

GNU GPL v3

See: [LICENSE](./LICENSE)

<!-- Named links -->
