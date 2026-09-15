# Molecule Test Runner

Run all Molecule scenarios and report results as a table.

## Instructions

1. Install dependencies (if not already present):

   ```bash
   ansible-galaxy collection install -r requirements.yml -p collections
   ```

2. For each platform, run each Molecule step individually
   to isolate failures. Since Molecule runs all platforms together,
   if a single platform fails (e.g., Alpine TLS in sandboxed
   environments), comment it out in `molecule.yml` temporarily and
   re-run the step for the remaining platforms:

   ```bash
   molecule destroy -s <scenario>
   molecule create -s <scenario>
   molecule converge -s <scenario>
   molecule idempotence -s <scenario>
   molecule verify -s <scenario>
   molecule destroy -s <scenario>
   ```

3. Record every step outcome (✅ pass, ❌ fail, ⏭️ skipped).

4. Report results in a **Scenario ✕ Platform** table (see template below).

## Scenarios

| Scenario | `wine_release` | Winetricks | Notes |
| --- | --- | --- | --- |
| `default` | `stable` | No | Version-pinned per host via `host_vars` |
| `devel` | `devel` | Yes | Development release + winetricks |
| `staging` | `staging` | Yes | Staging release + winetricks |
| `winetricks` | (default) | Yes | Winetricks install path only |

## Platforms (per scenario)

Platform names follow the `<role>-<scenario>-<platform>` convention, so each
scenario gets its own containers. For the `default` scenario:

| Container | Image | Notes |
| --- | --- | --- |
| `wine-default-alpine-latest` | `alpine:3.20` | Uses apk; Wine from Alpine repos |
| `wine-default-debian-latest` | `debian:latest` | WineHQ apt repo; codename `bookworm` |
| `wine-default-nixos-latest` | `nixos/nix:latest` | Custom Dockerfile; privileged mode |
| `wine-default-ubuntu-jammy` | `ubuntu:jammy` | WineHQ repo; codename `jammy` |
| `wine-default-ubuntu-noble` | `ubuntu:noble` | WineHQ repo; codename `jammy` |

The other scenarios (`devel`, `staging`, `winetricks`) use the same platform
suffixes with their own scenario segment, e.g. `wine-devel-debian-latest`. This
keeps containers unique across roles and scenarios, because Molecule's Docker
driver names each container exactly after its platform. Generic names such as
`debian-latest` would collide with concurrent Molecule runs of other roles.

## Results Template

Fill in each cell after running the tests.
Use ✅ for pass, ❌ for fail, ⏭️ for skipped.

### Step-Level Results (per scenario)

For each scenario, report per-platform step results:

| Platform | create | prepare | converge | idempotence | verify |
| --- | :---: | :---: | :---: | :---: | :---: |
| `wine-<scenario>-alpine-latest` | | | | | |
| `wine-<scenario>-debian-latest` | | | | | |
| `wine-<scenario>-nixos-latest` | | | | | |
| `wine-<scenario>-ubuntu-jammy` | | | | | |
| `wine-<scenario>-ubuntu-noble` | | | | | |

### Summary (all scenarios)

Rows are the platform suffix; the container for a given cell is
`wine-<scenario>-<platform>`.

| Platform | default | devel | staging | winetricks |
| --- | :---: | :---: | :---: | :---: |
| `alpine-latest` | | | | |
| `debian-latest` | | | | |
| `nixos-latest` | | | | |
| `ubuntu-jammy` | | | | |
| `ubuntu-noble` | | | | |

## Troubleshooting

- If NixOS fails with SSL errors, check `Dockerfile.j2` CA cert injection.
- If Wine GPG key download fails, verify `dl.winehq.org` is reachable.
- If winetricks download fails, verify `raw.githubusercontent.com` is reachable.
- If pip fails inside molecule-action, ensure `create.yml`/`destroy.yml`
  use `ansible.builtin.command` instead of `ansible.builtin.pip`.
- Refer to [AGENTS.md](../../AGENTS.md) for the full troubleshooting matrix.
