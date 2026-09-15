# Molecule Testing

## Molecule Scenarios

| Scenario | `wine_release` | Winetricks | Notes |
| -------- | -------------- | ---------- | ----- |
| `default` | `stable` | No | Version-pinned per host via `host_vars` |
| `devel` | `devel` | Yes | Tests development release + winetricks |
| `staging` | `staging` | Yes | Tests staging release + winetricks |
| `winetricks` | (default) | Yes | Tests winetricks install path |

### Platforms (per scenario)

Platform names follow the `<role>-<scenario>-<platform>` convention, so each
scenario gets its own containers. For the `default` scenario:

| Container | Image | Notes |
| --------- | ----- | ----- |
| `wine-default-alpine-latest` | `alpine:3.20` | Uses apk; Wine from Alpine repos |
| `wine-default-debian-latest` | `debian:latest` | Uses WineHQ apt repo |
| `wine-default-nixos-latest` | `nixos/nix:latest` | Custom Dockerfile; privileged mode |
| `wine-default-ubuntu-jammy` | `ubuntu:jammy` | WineHQ repo with `wine_release_codename: jammy` |
| `wine-default-ubuntu-noble` | `ubuntu:noble` | WineHQ repo with `wine_release_codename: jammy` |

The other scenarios (`devel`, `staging`, `winetricks`) use the same platform
suffixes with their own scenario segment, e.g. `wine-devel-debian-latest`. This
keeps containers unique across roles and scenarios, because Molecule's Docker
driver names each container exactly after its platform. Generic names such as
`debian-latest` would collide with concurrent Molecule runs of other roles.

### Running Tests

If asked to run Molecule tests, MUST follow the instructions in
[.github/prompts/molecule-test.prompt.md](../.github/prompts/molecule-test.prompt.md).

Molecule and Ansible are installed via the project `Pipfile`, so run every command through `pipenv`
(they are not on `PATH`).

```bash
# Full test (all scenarios)
pipenv run molecule test

# Single scenario
pipenv run molecule test -s default

# Individual steps
pipenv run molecule create -s default
pipenv run molecule converge -s default
pipenv run molecule verify -s default
pipenv run molecule destroy -s default

# Syntax check only
pipenv run molecule syntax
```

### Sandboxed / firewalled environments

In sandboxed environments (e.g. no outbound NAT on the default Docker bridge, or a resolver that
returns non-routable IPv6 addresses), opt in to the following environment variables:

- `MOLECULE_DOCKER_NETWORK=host` - run containers and image builds on the host network; required
  when the default bridge has no outbound NAT.
- `MOLECULE_DOCKER_FORCE_IPV4=true` - prefer IPv4 for DNS resolution inside containers; required
  when the resolver returns IPv6 addresses that are not routable.

Example invocation:

```bash
MOLECULE_DOCKER_NETWORK=host MOLECULE_DOCKER_FORCE_IPV4=true pipenv run molecule test -s default
```

## Molecule Gates

- `pipenv run molecule syntax` - YAML + playbook syntax validation
- `pipenv run molecule converge` - full role execution on all containers
- `pipenv run molecule idempotence` - re-run must produce zero changes
- `pipenv run molecule verify` - asserts `wine --version` succeeds + winetricks if enabled

## Troubleshooting Matrix

### Molecule prepare fails with DNS resolution errors

> `Temporary failure resolving 'deb.debian.org'` (or `azure.archive.ubuntu.com`), followed by
> `E: Unable to locate package python3` during the `prepare` step.

- **Root cause**: `docker0` has lost the `bridge` network's configured gateway address, so containers
  on the default bridge have no working gateway and cannot resolve DNS or reach the network.
- **Check**: `ip -4 addr show docker0` vs
  `docker network inspect bridge --format '{{range .IPAM.Config}}{{.Gateway}}{{end}}'`.
  If the gateway address is missing from `docker0`, this is the cause.
- **Fix (durable)**: `sudo systemctl restart docker` recreates `docker0` with the configured gateway.
  This restarts the daemon and stops any running containers.
- **Fix (non-disruptive, not persistent)**: `sudo ip addr add 172.17.0.1/16 dev docker0`
  (use the gateway reported by the check above).
- **Workaround (no sudo)**: run the tests on the host network, which bypasses the broken bridge.

```bash
MOLECULE_DOCKER_NETWORK=host \
MOLECULE_DOCKER_FORCE_IPV4=true \
MOLECULE_XVFB_DISPLAY_BASE=90 \
pipenv run molecule test
```

### NixOS container build fails with SSL/channel errors

- Root cause: `nix-channel --update` inside Docker can fail with sandbox or SSL issues
- Isolation: Check `molecule/resources/playbooks/Dockerfile.j2`
- Fix: Dockerfile injects proxy CA certs into Nix cert bundle via `ssl-cert-file`
  in `nix.conf`; cert setup and `nix-channel --update` are in a single `RUN` layer
- Required hosts: `channels.nixos.org`, `releases.nixos.org`, `cache.nixos.org`
  (channels.nixos.org redirects to releases.nixos.org; cache.nixos.org serves binaries)
- Prevention: All three Nix hosts must be in firewall allowlist

### NixOS: files in `/etc/ssl/certs/` vanish across Docker build layers

- Root cause: containerd/overlayfs bug causes files written to `/etc/ssl/certs/`
  in one Docker `RUN` layer to disappear in subsequent layers (NixOS image only)
- Isolation: `docker build` with separate RUN steps writing + reading a file there
- Fix: Store combined CA bundle in `/etc/nix/ca-bundle.crt` instead;
  `/etc/nix/` persists correctly across layers
- Prevention: NEVER store persistent files under `/etc/ssl/certs/` in NixOS containers

### `community.docker.docker_container` not found during molecule run

- Root cause: Collections installed to `./collections` but not on Ansible's search path.
  Molecule runs `ansible-playbook` with the **scenario directory** as CWD
  (`molecule/<scenario>/`), so a *relative* `ANSIBLE_COLLECTIONS_PATH` does not resolve.
  The env var also takes precedence over `collections_path` in Molecule's generated
  `ansible.cfg`, so the absolute path configured there is bypassed.
- Isolation: `molecule destroy -s <scenario>` fails at the first `ansible-playbook` call
  with `couldn't resolve module/action 'community.docker.docker_container'`.
- Fix: `collections_path` in molecule `config_options.defaults` includes `./collections`
  (resolved against `MOLECULE_PROJECT_DIRECTORY`), and CI sets an **absolute**
  `ANSIBLE_COLLECTIONS_PATH: ${{ github.workspace }}/collections`.
- Prevention: Never set `ANSIBLE_COLLECTIONS_PATH` to a relative path; all scenario
  configs MUST include `collections_path`.

### `allow_broken_conditionals` errors on molecule-docker 2.1.0

- Root cause: molecule-docker uses non-boolean `when:` in create/destroy playbooks
- Fix: Set `allow_broken_conditionals: true` in all scenario configs
- Ref: <https://github.com/ansible-community/molecule-plugins/issues/311>

### GitHub Actions Molecule report step fails with summary size limit

- **Root cause**: GitHub job summaries are capped at 1 MiB, but full Molecule HTML-to-Markdown conversions can exceed it.
- **Fix**: Upload full Molecule HTML reports as workflow artifacts and append only a concise filtered summary
  (e.g., Play Recap, errors, and warnings) to `$GITHUB_STEP_SUMMARY`.

### Molecule report `EACCES: permission denied`

- **Root cause**: The report file generated by `gofrolist/molecule-action` is owned by root with restricted permissions
  because it is created inside a Docker container.
- **Fix**: Run `sudo chown "$USER":"$USER" "$REPORT" || true` on the report file before attempting to read it
  (for summary) or upload it.

### Converge fails on wrong Wine version format for a platform

- Root cause: Debian-style version strings (e.g. `10.0.0.0~jammy-1`) applied to Alpine/NixOS
- Fix: Use `host_vars` per platform in molecule.yml; keep `converge.yml` generic
- NEVER set platform-specific vars as role vars in `converge.yml`
