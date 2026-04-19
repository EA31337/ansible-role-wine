# AGENTS.md

Persistent context for autonomous agents working on this Ansible role.

For project overview and install instructions, see [README.md](README.md).

## Setup & Environment Invariants

- Ansible role: `ea31337.wine`
- Supported OS: Alpine Linux, Debian/Ubuntu, NixOS (Nix)
- Driver: Docker (Molecule)
- Python required on all targets
- Collections: `community.docker`, `community.general`
- Vars: `defaults/main.yml` (user-facing), `vars/main.yml` (internal)

## Key Files & Context Injection

| Path | Purpose |
| ---- | ------- |
| `defaults/main.yml` | Default role variables (public API) |
| `vars/main.yml` | Internal variables (become methods, package maps) |
| `tasks/main.yml` | Role entry point |
| `tasks/wine/wine.yml` | OS-family dispatcher for Wine install |
| `tasks/wine/wine-Alpine.yml` | Alpine-specific Wine install |
| `tasks/wine/wine-Debian.yml` | Debian/Ubuntu Wine install (WineHQ repo) |
| `tasks/wine/wine-NixOS.yml` | Nix-based Wine install |
| `tasks/winetricks/winetricks.yml` | Winetricks dispatcher |
| `tasks/verify.yml` | Post-install verification tasks |
| `molecule/default/molecule.yml` | Default Molecule scenario config |
| `molecule/default/converge.yml` | Converge playbook (all scenarios) |
| `molecule/default/create.yml` | Custom Docker create playbook |
| `molecule/default/destroy.yml` | Custom Docker destroy playbook |
| `molecule/default/prepare.yml` | Container preparation (sudo, Python) |
| `molecule/default/verify.yml` | Verification playbook |
| `molecule/resources/playbooks/Dockerfile.j2` | NixOS container Dockerfile template |
| `templates/winehq.Debian.list.j2` | WineHQ apt source (Debian) |
| `templates/winehq.Ubuntu.list.j2` | Ubuntu backports apt source |
| `requirements.yml` | Galaxy collection requirements |
| `.github/workflows/molecule.yml` | CI: Molecule test matrix |
| `.github/workflows/check.yml` | CI: pre-commit / linting |

## Agent Directives

- MUST run `yamllint .` and `ansible-lint` before committing YAML changes.
- MUST keep YAML keys sorted alphabetically where noted in file headers.
- MUST use FQCN for all modules (e.g. `ansible.builtin.command`, not `command`).
- MUST enforce max line length of 120 characters (`.yamllint`, `.markdownlint.yaml`).
- MUST ensure idempotency in all Ansible tasks.
- MUST update `defaults/main.yml` and `README.md` when changing role variables.
- NEVER hardcode secrets or environment-specific values.
- NEVER remove or modify tests to mask failures; fix root cause instead.
- NEVER use `git add .` without verifying staged files.

## Molecule Scenarios

| Scenario | `wine_release` | Winetricks | Notes |
| -------- | -------------- | ---------- | ----- |
| `default` | `stable` | No | Version-pinned per host via `host_vars` |
| `devel` | `devel` | Yes | Tests development release + winetricks |
| `staging` | `staging` | Yes | Tests staging release + winetricks |
| `winetricks` | (default) | Yes | Tests winetricks install path |

### Platforms (all scenarios)

| Container | Image | Notes |
| --------- | ----- | ----- |
| `alpine-latest` | `alpine:3.20` | Uses apk; Wine from Alpine repos |
| `debian-latest` | `debian:latest` | Uses WineHQ apt repo |
| `nixos-latest` | `nixos/nix:latest` | Custom Dockerfile; privileged mode |
| `ubuntu-jammy` | `ubuntu:jammy` | WineHQ repo with `wine_release_codename: jammy` |
| `ubuntu-noble` | `ubuntu:noble` | WineHQ repo with `wine_release_codename: jammy` |
| `ubuntu-latest` | `ubuntu:latest` | WineHQ repo with `wine_release_codename: jammy` |

### Running Tests

```bash
# Full test (all scenarios)
molecule test

# Single scenario
molecule test -s default

# Individual steps
molecule create -s default
molecule converge -s default
molecule verify -s default
molecule destroy -s default

# Syntax check only
molecule syntax
```

## Testing & Verification Gates

- `molecule syntax` — YAML + playbook syntax validation
- `molecule converge` — full role execution on all containers
- `molecule idempotence` — re-run must produce zero changes
- `molecule verify` — asserts `wine --version` succeeds + winetricks if enabled
- `yamllint .` — YAML lint (config: `.yamllint`)
- `ansible-lint` — Ansible best practices (config: `.ansible-lint`)
- `pre-commit run -a` — all pre-commit hooks

## Troubleshooting Matrix

> NixOS container build fails with SSL/channel errors

- Root cause: `nix-channel --update` inside Docker can fail with sandbox or SSL issues
- Isolation: Check `molecule/resources/playbooks/Dockerfile.j2`
- Fix: Dockerfile injects proxy CA certs into Nix cert bundle via `ssl-cert-file`
  in `nix.conf`; cert setup and `nix-channel --update` are in a single `RUN` layer
- Required hosts: `channels.nixos.org`, `releases.nixos.org`, `cache.nixos.org`
  (channels.nixos.org redirects to releases.nixos.org; cache.nixos.org serves binaries)
- Prevention: All three Nix hosts must be in firewall allowlist

> NixOS: files in `/etc/ssl/certs/` vanish across Docker build layers

- Root cause: containerd/overlayfs bug causes files written to `/etc/ssl/certs/`
  in one Docker `RUN` layer to disappear in subsequent layers (NixOS image only)
- Isolation: `docker build` with separate RUN steps writing + reading a file there
- Fix: Store combined CA bundle in `/etc/nix/ca-bundle.crt` instead;
  `/etc/nix/` persists correctly across layers
- Prevention: NEVER store persistent files under `/etc/ssl/certs/` in NixOS containers

> `community.docker.docker_container` not found during molecule run

- Root cause: Collections installed to `./collections` but not on Ansible search path
- Fix: `collections_path` in molecule `config_options.defaults` includes `./collections`
- Prevention: All scenario configs MUST include `collections_path`

> Wine GPG key download fails (`dl.winehq.org` unreachable)

- Root cause: Firewall/network policy blocks `dl.winehq.org`
- Fix: Add `dl.winehq.org` to firewall allowlist
- Prevention: Keep firewall rules documented in `.github/agents/FIREWALL.md`

> Alpine apk fails with SSL certificate errors

- Root cause: HTTPS interception or missing CA certs in container
- Isolation: `docker run --rm alpine:3.20 apk update`
- Fix: Ensure CA certificates are present; check proxy/firewall SSL inspection

> `allow_broken_conditionals` errors on molecule-docker 2.1.0

- Root cause: molecule-docker uses non-boolean `when:` in create/destroy playbooks
- Fix: Set `allow_broken_conditionals: true` in all scenario configs
- Ref: <https://github.com/ansible-community/molecule-plugins/issues/311>

> Converge fails on wrong Wine version format for a platform

- Root cause: Debian-style version strings (e.g. `10.0.0.0~jammy-1`) applied to Alpine/NixOS
- Fix: Use `host_vars` per platform in molecule.yml; keep `converge.yml` generic
- NEVER set platform-specific vars as role vars in `converge.yml`

## Linting and Validation

```bash
# Run all pre-commit checks
pre-commit run -a

# Individual linters
yamllint .
ansible-lint
```

## Common Tasks

### Before Each Commit

- Verify changes with `git diff --no-color`.
- Ensure no temporary or unrelated files are staged.
- Run `yamllint .` and `ansible-lint` for any YAML changes.
- Run `molecule syntax` to catch playbook errors early.

### Editing Files

- Enforce line-wrapping per `.markdownlint.yaml` (120 chars) and `.yamllint` (120 chars).
- Keep YAML keys alphabetically sorted where file headers indicate.

### Renaming/Removing Files

- Use `git mv` / `git rm` to preserve history.

## Firewall Issues

If network requests fail during molecule tests (e.g. `dl.winehq.org`,
`channels.nixos.org`, `galaxy.ansible.com`):

- Refer to <https://gh.io/copilot/firewall-config> for agent firewall setup.
- Do not work around blocked URLs; request allowlisting instead.
- Document required hosts in `.github/agents/FIREWALL.md`.

### Required Hosts

| Host | Purpose |
| ---- | ------- |
| `dl.winehq.org` | WineHQ APT repository and GPG key |
| `channels.nixos.org` | Nix channel metadata (redirects to releases.nixos.org) |
| `releases.nixos.org` | Nix channel tarballs (redirect target) |
| `cache.nixos.org` | Nix binary cache (pre-built packages) |
| `galaxy.ansible.com` | Ansible Galaxy collections |
| `raw.githubusercontent.com` | Winetricks script download |

## References

- Project documentation: [README.md](README.md)
- Agent configuration: [.github/copilot-instructions.md](.github/copilot-instructions.md)
- Org baseline: <https://github.com/Cogni-AI-OU/.github/blob/main/AGENTS.md>
- Agents.md standard: <https://agents.md/>
