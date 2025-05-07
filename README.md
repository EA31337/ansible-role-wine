# Ansible Role: Wine

[![CodeRabbit PR Reviews](https://img.shields.io/coderabbit/prs/github/EA31337/ansible-role-wine?utm_source=oss&utm_medium=github&utm_campaign=EA31337%2Fansible-role-wine&labelColor=171717&color=FF570A&link=https%3A%2F%2Fcoderabbit.ai&label=CodeRabbit+PR+Reviews)](https://github.com/EA31337/ansible-role-wine/pulls)
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

## Testing

### Docker

To test on Docker containers, run:

```shell
ansible-playbook -i tests/inventory/docker-containers.yml tests/playbooks/docker-containers.yml
```

### Molecule

To test using Molecule, run:

```shell
molecule test
```
