# Ansible Role: Wine

[![License][license-badge]][license-link]
[![Check][check-badge]][check-link]
[![Dev][dev-badge]][dev-link]
[![Molecule][molecule-badge]][molecule-link]
[![Pull Requests][pr-badge]][pr-link]
[![Test][test-badge]][test-link]
[![Edit][gh-edit-badge]][gh-edit-link]

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

Available variables are listed below, along with default values (see [defaults/main.yml][defaults-link]):

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

# Whether to initialize ~/.wine (skipped in Docker)
wine_init: false
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

## Docker Image

The Docker image is automatically built and pushed to the GitHub Container Registry (GHCR) on every merge to `master`
and on new tags.

### Pulling the Image

To pull the latest image:

```shell
docker pull ghcr.io/ea31337/ansible-role-wine:latest
```

### Available Tags

The following tags are available for different platforms:

| Platform | Latest Tag | Branch Tag | Version Tag Example |
| -------- | ---------- | ---------- | ------------------- |
| **Ubuntu Latest** (Default) | `latest` | `master` | `v1.0.0` |
| Ubuntu Noble (24.04) | `ubuntu-noble` | `latest-ubuntu-noble` | `v1.0.0-ubuntu-noble` |
| Ubuntu Jammy (22.04) | `ubuntu-jammy` | `latest-ubuntu-jammy` | `v1.0.0-ubuntu-jammy` |
| Debian Latest | `debian-latest` | `latest-debian-latest` | `v1.0.0-debian-latest` |
| Alpine Latest | `alpine-latest` | `latest-alpine-latest` | `v1.0.0-alpine-latest` |
| NixOS Latest | `nixos-latest` | `latest-nixos-latest` | `v1.0.0-nixos-latest` |

For a specific branch (e.g., `develop`), use the branch name as the tag: `ghcr.io/ea31337/ansible-role-wine:develop-<platform>`.

## License

GNU GPL v3

See: [LICENSE][license-link]

<!-- Named links -->

[license-badge]: https://img.shields.io/badge/license-GPLv3-brightgreen.svg
[license-link]: ./LICENSE
[check-badge]: https://img.shields.io/github/actions/workflow/status/EA31337/ansible-role-wine/check.yml?label=Check
[check-link]: https://github.com/EA31337/ansible-role-wine/actions/workflows/check.yml
[dev-badge]: https://img.shields.io/github/actions/workflow/status/EA31337/ansible-role-wine/devcontainer-ci.yml?label=Dev
[dev-link]: https://github.com/EA31337/ansible-role-wine/actions/workflows/devcontainer-ci.yml
[molecule-badge]: https://img.shields.io/github/actions/workflow/status/EA31337/ansible-role-wine/molecule.yml?label=Molecule
[molecule-link]: https://github.com/EA31337/ansible-role-wine/actions/workflows/molecule.yml
[pr-badge]: https://img.shields.io/github/issues-pr/EA31337/ansible-role-wine.svg
[pr-link]: https://github.com/EA31337/ansible-role-wine/pulls
[test-badge]: https://img.shields.io/github/actions/workflow/status/EA31337/ansible-role-wine/test.yml?label=Test
[test-link]: https://github.com/EA31337/ansible-role-wine/actions/workflows/test.yml
[gh-edit-badge]: https://img.shields.io/badge/GitHub-edit-purple.svg?logo=github
[gh-edit-link]: https://github.dev/EA31337/ansible-role-wine
[defaults-link]: defaults/main.yml
