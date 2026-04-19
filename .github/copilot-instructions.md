# Copilot Instructions for ansible-role-wine

## Project Overview

Ansible role (`ea31337.wine`) to install and configure Wine on UNIX-like
platforms with distribution-specific approaches:

- **Alpine Linux**: Uses apk package manager
- **Debian/Ubuntu**: Uses apt package manager with WineHQ repository
- **NixOS / Nix**: Uses nix-env in lightweight Nix environments

Key contents:

- **Role entry point**: `tasks/main.yml` dispatches to OS-specific task files
- **OS task files**: `tasks/wine/wine-Alpine.yml`, `wine-Debian.yml`, `wine-NixOS.yml`
- **Variables**: `defaults/main.yml` (public API), `vars/main.yml` (internal)
- **Templates**: Jinja2 templates for WineHQ apt sources
- **Molecule tests**: Docker-based integration tests across all supported platforms

### Getting Started

- Refer to `README.md` for installation and usage instructions.
- Refer to `AGENTS.md` for persistent agent context, troubleshooting, and
  molecule scenario details.
- Check `.devcontainer/devcontainer.json` for the Codespaces/devcontainer setup.

You are expected to be an expert in:

- Ansible
- Python
- Jinja2
- Molecule
- Linux (Alpine, Debian/Ubuntu, Nix)
- YAML

## Code Standards

- Avoid writing trailing whitespace
- Follow PEP 8 for Python
- Include docstrings and type hints where applicable
- Maintain consistent YAML indentation
- Optimize for readability first, performance second
- Prefer modular, DRY approaches and list comprehensions when appropriate
- Use environment variables for configuration, never hardcode sensitive info
- Write clean, documented, error-handling code with appropriate logging

## General Approach

- Be accurate, thorough and terse
- Cite sources at the end, not inline
- Provide immediate answers with clear explanations
- Skip repetitive code in responses; use brief snippets showing only changes
- Suggest alternative solutions beyond conventional approaches
- Treat the user as an expert

## Ansible Guidelines

- Ensure idempotency in all tasks
- Ensure indentation is correct, especially for YAML files
- Follow standard role structure: tasks/, handlers/, templates/, defaults/, meta/
- Use ansible-lint and write Molecule tests for verification
- Use descriptive task names and include helpful comments

### Ansible Linting

Ensure enforcing the following rules:

- fqcn[keyword]: Avoid `collections` keyword by using FQCN for all plugins,
  modules, roles and playbooks

## Formatting Guidelines

### YAML

Follow the YAML rules defined in `.yamllint`:

- yaml[empty-lines]: Avoid too many blank lines
- yaml[indentation]: Avoid wrong indentation
- yaml[line-length]: No long lines (max. 120 characters)
- yaml[new-line-at-end-of-file]: Enforce new line character at the end of file
- yaml[truthy]: Truthy value should be one of [false, true]
- Ensure items are in lexicographical order when possible
- When writing inline code, add a new line at the end to maintain proper
  indentation

To verify locally, run `yamllint .` or `pre-commit run yamllint -a`.

### Markdown

Follow the Markdown rules defined in `.markdownlint.yaml`:

- MD013: Line length max 120 characters
- MD033: Inline HTML allowed only for `<details>` elements
- MD046: Consistent code block style

To verify locally, run `pre-commit run markdownlint -a`.

## Project Structure

```text
.
├── .devcontainer/            # Codespaces / devcontainer configuration
├── .github/
│   ├── prompts/              # Copilot prompt files
│   ├── workflows/            # GitHub Actions (molecule, check, test)
│   └── copilot-instructions.md
├── defaults/
│   └── main.yml              # Default role variables (public API)
├── meta/
│   └── main.yml              # Role metadata (Galaxy, dependencies)
├── molecule/
│   ├── default/              # Default Molecule scenario
│   │   ├── molecule.yml      # Scenario config (platforms, provisioner)
│   │   ├── converge.yml      # Converge playbook
│   │   ├── create.yml        # Custom Docker create (proxy CA injection)
│   │   ├── destroy.yml       # Custom Docker destroy
│   │   ├── prepare.yml       # Container preparation (sudo, Python, certs)
│   │   └── verify.yml        # Verification playbook
│   └── resources/playbooks/
│       └── Dockerfile.j2     # NixOS container Dockerfile template
├── tasks/
│   ├── main.yml              # Role entry point
│   └── wine/
│       ├── wine.yml          # OS-family dispatcher
│       ├── wine-Alpine.yml   # Alpine-specific Wine install
│       ├── wine-Debian.yml   # Debian/Ubuntu Wine install (WineHQ repo)
│       └── wine-NixOS.yml    # Nix-based Wine install
├── templates/                # Jinja2 templates (apt sources)
├── vars/
│   └── main.yml              # Internal variables (become methods, pkg maps)
├── AGENTS.md                 # AI agent guidance and troubleshooting
├── README.md                 # Project documentation
└── requirements.yml          # Galaxy collection requirements
```

### Key Variables

Variables are defined in `defaults/main.yml` (user-facing) and
`vars/main.yml` (internal).

Notes:

- On variable changes, update `defaults/main.yml` and `README.md` accordingly.

### Devcontainer Guidance

Project utilizes Codespaces with config at `.devcontainer/devcontainer.json`
and requirements at `.devcontainer/requirements.txt`.

- Treat the repository devcontainer as the default controller environment.
- Keep controller dependency installation in the devcontainer configuration
  so Molecule scenarios can assume those tools are already available.
- If dependencies are missing, update `.devcontainer/requirements.txt` or
  `.devcontainer/requirements-ansible.txt` instead of adding per-run install
  steps.

## Testing Approach

- Use Molecule with Docker driver
- GitHub Actions run pre-commit checks (`.pre-commit-config.yaml`) and
  Molecule (`molecule/`)
- Service management uses supervisord across platforms
- Formatting rules: `.yamllint` (YAML) and `.markdownlint.yaml` (Markdown)

### Running Tests

```bash
# Full molecule test (all scenarios)
molecule test

# Single scenario
molecule test -s default

# Individual steps
molecule create -s default
molecule converge -s default
molecule verify -s default
molecule destroy -s default

# Linting
yamllint .
ansible-lint
pre-commit run -a
```

### Platforms

| Container | Image | Notes |
| --------- | ----- | ----- |
| `alpine-latest` | `alpine:3.20` | Uses apk; Wine from Alpine repos |
| `debian-latest` | `debian:latest` | WineHQ apt repo; codename: `bookworm` |
| `nixos-latest` | `nixos/nix:latest` | Custom Dockerfile; privileged mode |
| `ubuntu-jammy` | `ubuntu:jammy` | WineHQ repo; codename: `jammy` |
| `ubuntu-noble` | `ubuntu:noble` | WineHQ repo; codename: `jammy` |
| `ubuntu-latest` | `ubuntu:latest` | WineHQ repo; codename: `jammy` |

## Troubleshooting

### Finding Build Errors

1. **Reproduce errors locally:**
   - For pre-commit errors: `pre-commit run -a`
   - For Molecule errors: `molecule test -s default`
   - For linting: `yamllint .` and `ansible-lint`

2. **Common error patterns:**
   - **NixOS SSL/channel errors**: Proxy CA certs must be injected via
      `Dockerfile.j2` and `prepare.yml`. Combined cert bundle is stored
      at `/etc/nix/ca-bundle.crt` (NOT `/etc/ssl/certs/` — files there
      vanish across Docker overlay layers in the NixOS image).

   - **NixOS firewall**: `channels.nixos.org`, `releases.nixos.org`, and
     `cache.nixos.org` must all be in the firewall allowlist.
   - **Wine GPG key failures**: `dl.winehq.org` must be in the firewall
     allowlist.
   - **`collections_path` not found**: Ensure `collections_path` includes
     `./collections` in all scenario configs.
   - **`allow_broken_conditionals`**: molecule-docker 2.1.0 requires
     `allow_broken_conditionals: true` in provisioner config.

3. **Refer to `AGENTS.md`** for the full troubleshooting matrix.
