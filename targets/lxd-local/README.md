# lxd-local managed nodes

Shared infrastructure, not an exercise itself. Unlike `docker-local` and
`vagrant-local`, this target isn't primarily about being a cheaper or more
faithful Linux host — Incus system containers sit between the two on that
axis (real systemd as PID 1 and real service management, unlike Docker; much
lighter than a VM, unlike Vagrant/libvirt). The actual point of this target
is the connection mechanism: Ansible can manage an Incus instance through a
native API-based connection plugin instead of SSH, which is a genuinely
different thing to understand than anything docker-local/vagrant-local
teach — every other target in this repo is "Ansible over SSH to a Linux
host"; this one is "Ansible over a provider's own API." Confirm the exact
collection and plugin name in-session with `ansible-doc -l | grep -i -E
'incus|lxd'` rather than assuming one — collection names and support move
between `community.general` and the dedicated `containers.incus` collection,
and the wrong guess here wastes a session. This file and the host-side setup
below are infrastructure Claude maintains, the same tier as `.devcontainer/`
— not the learning material itself. Everything inside an `lxd-local/<slug>/`
exercise directory (the playbook, roles, inventory/connection config, that
exercise's own `ansible.cfg`) is written by you, from scratch, guided
Socratically, same as every other target (see CLAUDE.md's "Hard
constraints").

## One-time setup: Incus on the host, not nested in the devcontainer

Running Incus nested inside the devcontainer needs a privileged container
and working cgroup/AppArmor delegation — fragile, and avoided here for the
same reason `docker-local` uses Docker-outside-of-Docker rather than
Docker-in-Docker. Instead, Incus runs on the host itself, and the
devcontainer holds only the `incus` client, talking to it remotely:

1. Install Incus on the host machine (not the devcontainer) — see
   [linuxcontainers.org/incus/docs/main/installing](https://linuxcontainers.org/incus/docs/main/installing/)
   for the current instructions for your host OS.
2. `incus admin init` on the host (minimal/default answers are fine for this
   repo's purposes).
3. Expose it to the network and create a trust token for the devcontainer to
   use: `incus config set core.https_address :8443` then
   `incus config trust add devcontainer` — this prints a one-time token.
4. From inside the devcontainer: `incus remote add devhost
   https://<host-address>:8443 --token <token>`, where `<host-address>` is
   whatever address reaches the host from inside the devcontainer's network
   (often `host.docker.internal`, otherwise the host's LAN IP — confirm
   which resolves before relying on it).

This is a real prerequisite outside the devcontainer, same spirit as
`vagrant-local`'s KVM requirement: unavailable until done once, not broken
by default for anyone who skips it.

## Spin up instances

```bash
incus launch images:debian/12 <slug>-node1 --remote devhost
incus launch images:rockylinux/9 <slug>-node2 --remote devhost
incus list --remote devhost
```

Whether the exercise connects via the native plugin (API-based, no SSH, no
inventory host keys) or over SSH to an address `incus list` reports is
itself part of what the exercise should decide and record — don't default
to copying `docker-local`'s SSH-based inventory shape without first asking
whether that's actually the point of this particular session.

## Tear down

```bash
incus delete --force <slug>-node1 <slug>-node2 --remote devhost
```

Every instance is ephemeral: destroyed at the end of the sitting that
created it, regardless of whether the exercise concluded (the exercise's
*files* persist; the running instance never needs to).
