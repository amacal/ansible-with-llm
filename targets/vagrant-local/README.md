# vagrant-local managed nodes

Shared infrastructure, not an exercise itself — the two generic VM definitions
every `vagrant-local/` exercise's managed nodes are built from: Debian-family
(`debian/Vagrantfile`) and RHEL-family (`rhel/Vagrantfile`), the same
cross-target pairing as `docker-local`'s two node images. A real VM (not a
container) is the point: a full init system, real reboot behavior, real
kernel-level state — things a container can't honestly simulate. This file
and the two Vagrantfiles are infrastructure Claude maintains, the same tier
as `.devcontainer/` — not the learning material itself. Everything inside a
`vagrant-local/<slug>/` exercise directory (the playbook, roles,
`inventory.ini`, that exercise's own `ansible.cfg`) is written by you, from
scratch, guided Socratically, same as `docker-local` (see CLAUDE.md's "Hard
constraints").

## Before the first use: enable KVM passthrough

This needs actual nested virtualization, which the default devcontainer
doesn't request (requesting it unconditionally would break container startup
on a host that can't provide it). One-time setup, only on a host that
actually supports it (bare-metal Linux with virtualization enabled in BIOS —
not guaranteed inside a cloud-hosted devcontainer backend or Docker Desktop's
own VM on macOS/Windows without nested-virt explicitly enabled there):

1. Confirm the host has it: `ls /dev/kvm` should exist before you start.
2. Add to `.devcontainer/devcontainer.json`'s top level:
   ```json
   "runArgs": ["--device=/dev/kvm", "--cap-add=NET_ADMIN"]
   ```
3. **Dev Containers: Rebuild Container.**

If `/dev/kvm` isn't available on the host at all, this target isn't usable
there — fall back to `docker-local`, which has no such requirement.

## Start libvirtd (once per container boot)

The devcontainer doesn't run systemd as PID 1, so libvirtd isn't started
automatically:

```bash
sudo virsh list >/dev/null 2>&1 || sudo libvirtd -d
```

## Spin up a node

```bash
cd targets/vagrant-local/debian   # or rhel
vagrant up --provider=libvirt
vagrant ssh-config                # HostName, Port, User, IdentityFile
```

Write those connection details into that exercise's own `inventory.ini` —
never into this directory. This directory describes the two shared VM
definitions; it says nothing about any one exercise's instance of them.

## Tear down

```bash
cd targets/vagrant-local/debian   # or rhel, whichever was brought up
vagrant destroy -f
```

Every VM is ephemeral: destroyed at the end of the sitting that created it,
regardless of whether the exercise concluded (the exercise's *files* persist;
the running VM never needs to).
