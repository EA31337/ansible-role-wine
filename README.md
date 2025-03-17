# Ansible Role: Wine

[![License](https://img.shields.io/badge/license-MIT-brightgreen.svg)](LICENSE)

Ansible role to install Wine on UNIX-like platforms.
Wine (originally an acronym for "Wine Is Not an Emulator")
is a compatibility layer capable of running Windows applications
on several UNIX-like operating systems.

## Requirements

- Ansible 2.9 or higher
- Supported platforms:
  - Alpine 3.12+
  - Debian 10+
  - Ubuntu 18.04+
  - NixOS

## Installation

To install this role, run the following terminal command:

```shell
ansible-galaxy install git+https://github.com/EA31337/ansible-role-wine.git
```

## Role Variables

For available variables, check [`defaults/main.yml`](defaults/main.yml).
