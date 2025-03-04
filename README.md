# ansible-role-wine

Ansible role to install Wine on UNIX-like platforms.

## Install

To install this role, you can use the following terminal command:

    ansible-galaxy install git+https://github.com/EA31337/ansible-role-wine.git

## Requirements

- Ansible 2.9 or higher
- Supported platforms: Ubuntu, Debian, Alpine

## Role Variables

Available variables are listed below, along with default values (see `defaults/main.yml`):

    wine_version: "stable"  # Wine version to install (stable or development)
    wine_arch: "amd64"      # Architecture (amd64 or i386)
    wine_install_winetricks: false  # Whether to install Winetricks
