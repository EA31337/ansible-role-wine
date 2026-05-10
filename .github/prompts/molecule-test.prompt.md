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

## Platforms (shared by all scenarios)

| Container | Image | Notes |
| --- | --- | --- |
| `alpine-latest` | `alpine:3.20` | Uses apk; Wine from Alpine repos |
| `debian-latest` | `debian:latest` | WineHQ apt repo; codename `bookworm` |
| `nixos-latest` | `nixos/nix:latest` | Custom Dockerfile; privileged mode |
| `ubuntu-jammy` | `ubuntu:jammy` | WineHQ repo; codename `jammy` |
| `ubuntu-noble` | `ubuntu:noble` | WineHQ repo; codename `jammy` |

## Results Template

Fill in each cell after running the tests.
Use ✅ for pass, ❌ for fail, ⏭️ for skipped.

### Step-Level Results (per scenario)

For each scenario, report per-platform step results:

| Platform | create | prepare | converge | idempotence | verify |
| --- | :---: | :---: | :---: | :---: | :---: |
| `alpine-latest` | | | | | |
| `debian-latest` | | | | | |
| `nixos-latest` | | | | | |
| `ubuntu-jammy` | | | | | |
| `ubuntu-noble` | | | | | |

### Summary (all scenarios)

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
