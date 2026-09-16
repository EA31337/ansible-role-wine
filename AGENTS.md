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
| `docs/FACTS.mmd` | Mindmap of role structure and knowledge |
| `docs/FLOWS.mmd` | Flowchart of role execution and testing |
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
| `.github/prompts/molecule-test.prompt.md` | Step-by-step Molecule test runner prompt |

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
- MUST reference GitHub Actions by simple major version tags (e.g. `actions/checkout@v6`),
  not pinned patch versions (e.g. `@v6.1.0`), so minor/patch updates apply automatically.

## Docker Tests

The standalone Docker test playbooks in `tests/`, how to run them via `pipenv`, and
their troubleshooting matrix live in [tests/AGENTS.md](tests/AGENTS.md).

## Molecule Testing

Molecule scenarios, the platform matrix, how to run the tests, and Molecule-specific
troubleshooting live in [molecule/AGENTS.md](molecule/AGENTS.md).

## Testing & Verification Gates

- `yamllint .` - YAML lint (config: `.yamllint`)
- `ansible-lint` - Ansible best practices (config: `.ansible-lint`)
- `pre-commit run -a` - all pre-commit hooks

## Troubleshooting Matrix

> Wine GPG key download fails (`dl.winehq.org` unreachable)

- Root cause: Firewall/network policy blocks `dl.winehq.org`
- Fix: Add `dl.winehq.org` to firewall allowlist
- Prevention: Keep firewall rules documented in `.github/FIREWALL.md`

> Alpine apk fails with SSL certificate errors

- Root cause: HTTPS interception or missing CA certs in container
- Isolation: `docker run --rm alpine:3.20 apk update`
- Fix: Ensure CA certificates are present; check proxy/firewall SSL inspection

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
- Run `pipenv run molecule syntax` to catch playbook errors early.

### Updating Pre-commit Hooks

Run `pre-commit autoupdate`, then `pre-commit run -a`. Revert any hook that breaks and file an issue for it.

Known blockers (as of the 2026-09 update):

- `ansible-lint` v26.8.0 declares `language_version: python3.14`. Without a Python 3.14
  interpreter, either keep the ref pinned or override the hook with `language_version: python3`.
- `pre-commit-hooks` v6.0.0 removed `check-byte-order-marker`; replace it with
  `fix-byte-order-marker`.
- `markdownlint-cli` v0.49.1 needs node >= 22.20 (its dev dependency `ava@8`). If the hook pins
  `language_version: 22.14.0`, the env fails to install; pin markdownlint-cli or bump the pinned node.
- `ansible-lint` + `community.docker`: a stale, empty
  `.ansible/collections/ansible_collections/community/docker` directory shadows the real collection
  and causes `couldn't resolve module/action 'community.docker.docker_container'`. Remove it.
- `additional_dependencies` with a version range must use the block form
  (`- ansible-core>=2.16,<2.21`); the inline flow form splits on the comma into separate
  requirements, and the no-space form trips ansible-lint's `yaml[commas]` rule.

`pre-commit run -a` can also surface pre-existing failures (e.g. `yamlfix`/`black` reformatting,
`flake8` violations) unrelated to the ref bump; CI lints only changed files, so file these separately.

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
- Document required hosts in `.github/FIREWALL.md`.

### Alpine bootstrap fails with TLS error

- **Root cause**: Alpine `apk update` fails with `TLS: unspecified error` when behind an SSL-intercepting proxy
  if the proxy CA is not in the build-time trust store.
- **Fix**: The custom `Dockerfile.j2` injects host CA certificates directly into `/etc/ssl/cert.pem`
  during the build phase so `apk` can fetch dependencies safely.
- **Prevention**: Verify `dl-cdn.alpinelinux.org` is reachable from inside the container.

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
