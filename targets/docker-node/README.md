# docker-local managed nodes

Shared infrastructure, not an exercise itself — the two generic images every
`docker-local/` exercise's managed nodes are built from: Debian-family
(`debian/Dockerfile`) and RHEL-family (`rhel/Dockerfile`, dnf/systemd-style),
kept side by side specifically for cross-target comparison — the same role
run against both forces precision about what Ansible's generic modules
(`package`, `service`) actually abstract away versus what's genuinely
host-specific. Each image is minimal on purpose: `sshd` + `python3` (Ansible's
own requirement on a managed node) + a passwordless-sudo `ansible` user for
privilege-escalation exercises. Nothing else is pre-installed — every package
an exercise needs, the exercise's own playbook installs.

The two kinds of SSH key are handled differently. The login key is never
baked in, so rebuilding is never needed just to rotate it — a public key is
mounted into the container as the `ansible` user's `authorized_keys` at
`docker run` time, read fresh every time the container starts. The host
keys, by contrast, are generated once at build time (by the
`openssh-server` package's install step on Debian, by `ssh-keygen -A` on
RHEL), so every container started from the same image presents the same
host key — recreating a container never changes it; only rebuilding the
image does.

This file and the two Dockerfiles are infrastructure Claude maintains, the
same tier as `.devcontainer/` — not the learning material itself. Everything
inside a `docker-local/<slug>/` exercise directory (the playbook, roles,
`inventory.ini`, that exercise's own `ansible.cfg`) is written by you,
guided Socratically — fresh, or carried over from one of your own earlier
exercises (see CLAUDE.md's "Exercise files"). Claude never writes or
completes any of it (see CLAUDE.md's "Hard constraints").

## Spin up a node

The devcontainer talks to the host's Docker daemon (Docker-outside-of-Docker),
which has two consequences. First, a `-v` bind-mount source is resolved on the
*host*, where `/workspaces/...` does not exist — Docker silently creates an
empty directory there instead, `authorized_keys` becomes a directory, and SSH
fails with `Permission denied (publickey)`. The mount must use the host-side
path of the workspace, looked up from the devcontainer's own mounts. Second,
`-p 0:22` publishes on the host's interfaces, not the devcontainer's
`127.0.0.1`, so the node is reached directly on its bridge IP at port 22
(the devcontainer and the node share Docker's default bridge network). The
published port is therefore unused by any connection from here, but it is
kept deliberately: it is what Docker reports as the node's SSH endpoint, so
anything that reads the daemon's port mapping (such as the
`community.docker.docker_containers` inventory plugin in `ssh` mode) sees a
`127.0.0.1`-plus-host-port address that is unreachable from the
devcontainer — a real Docker-outside-of-Docker pitfall worth meeting. An
exercise that groups nodes by metadata can also add `--label key=value` to
`docker run` (e.g. `--label os_family=debian`); labels show up under
`.Config.Labels` in `docker inspect`.

```bash
# once per sitting, if not already generated:
ssh-keygen -t ed25519 -f .tmp/ansible_node_key -N "" -q

# host-side path of this workspace (run from the repo root):
HOST_WS=$(docker inspect "$(hostname)" \
  --format '{{range .Mounts}}{{if eq .Destination "'"$PWD"'"}}{{.Source}}{{end}}{{end}}')

docker build -t ansible-node-debian targets/docker-node/debian
docker run -d --name <exercise-dir>-node1 \
  -v "$HOST_WS/.tmp/ansible_node_key.pub:/home/ansible/.ssh/authorized_keys:ro" \
  -p 0:22 ansible-node-debian
# or targets/docker-node/rhel for the RHEL-family image

# confirm the key mounted as a file, not a directory:
docker exec <exercise-dir>-node1 ls -l /home/ansible/.ssh/

# the address to connect to (port 22):
docker inspect -f '{{.NetworkSettings.IPAddress}}' <exercise-dir>-node1
```

A node that has already been through a few runs no longer shows what a
first run does, because everything the playbook installs is already there.
So an exercise's from-scratch claim (it converges a fresh node, then a
second run reports zero changed) is checked against a recreated container
rather than a reused one. Recreating means removing the container and running
it again with the same name and the same `--label` flags, so dynamic-inventory
grouping stays the same. The host key stays the same as well, since it comes
from the image. The bridge IP usually comes back unchanged, but nothing
guarantees it:

```bash
docker rm -f <exercise-dir>-node1
docker run -d --name <exercise-dir>-node1 --label os_family=debian \
  -v "$HOST_WS/.tmp/ansible_node_key.pub:/home/ansible/.ssh/authorized_keys:ro" \
  -p 0:22 ansible-node-debian
```

The bridge IP, the key path, and every other connection detail belong in
that exercise's own `inventory.ini` and `ansible.cfg` — never here. The IP is
ephemeral (valid only for that container's lifetime), so an exercise's
inventory is valid only for the sitting that started its node. This file
describes the two shared images; it says nothing about any one exercise's
instance of them.

## Tear down

```bash
docker rm -f <exercise-dir>-node1 [<exercise-dir>-node2 ...]
ssh-keygen -R <node-bridge-ip>   # forget the IP-to-host-key pairing in ~/.ssh/known_hosts
```

Every node is ephemeral: destroyed at the end of the sitting that created it,
regardless of whether the exercise concluded (the exercise's *files* persist;
the running container never needs to). `.tmp/ansible_node_key*` is gitignored
and regenerated per sitting like any other `.tmp/` scratch content.

The `ssh-keygen -R` step matters because a bridge IP is reused by whatever
container starts next, possibly one from the other image with a different
host key; removing the entry turns that into an unknown key rather than a
changed one. Either way the next connection needs trust established again —
how an exercise does that (and whether it keeps its own known-hosts file
instead of `~/.ssh/known_hosts`) is that exercise's own decision.
