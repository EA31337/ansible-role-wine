# Ansible Role: Wine

[![License](https://img.shields.io/badge/license-MIT-brightgreen.svg)](LICENSE)

Ansible role to install Wine on UNIX-like platforms.
Wine (originally an acronym for "Wine Is Not an Emulator")
is a compatibility layer capable of running Windows applications
on several UNIX-like operating systems.

## Requirements

- Ansible 2.9 or higher
- Supported platforms:
  - Ubuntu 18.04+
  - Debian 10+
  - Alpine 3.12+

## Installation

To install this role, you can use the following terminal command:

```shell
ansible-galaxy install git+https://github.com/EA31337/ansible-role-wine.git
```

## Role Variables

Available variables are listed below,
along with default values (see `defaults/main.yml`):

```yaml
    # Wine release to install.
    wine_release: stable
    # Whether to install Winetricks.
    wine_winetricks_apt_install: false
```
