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

Neither image bakes in an SSH key at build time, so rebuilding is never
needed just to rotate keys — a public key is mounted into the container at
`docker run` time instead, read fresh every time the container starts.

This file and the two Dockerfiles are infrastructure Claude maintains, the
same tier as `.devcontainer/` — not the learning material itself. Everything
inside a `docker-local/<slug>/` exercise directory (the playbook, roles,
`inventory.ini`, that exercise's own `ansible.cfg`) is written by you, from
scratch, guided Socratically — same as every `.rs`/`.c` file in
math-with-llm/hard-with-llm. Claude never writes or completes any of it (see
CLAUDE.md's "Hard constraints").

## Spin up a node

The devcontainer talks to the host's Docker daemon (Docker-outside-of-Docker),
which has two consequences. First, a `-v` bind-mount source is resolved on the
*host*, where `/workspaces/...` does not exist — Docker silently creates an
empty directory there instead, `authorized_keys` becomes a directory, and SSH
fails with `Permission denied (publickey)`. The mount must use the host-side
path of the workspace, looked up from the devcontainer's own mounts. Second,
`-p 0:22` publishes on the host's interfaces, not the devcontainer's
`127.0.0.1`, so the node is reached directly on its bridge IP at port 22
(the devcontainer and the node share Docker's default bridge network).

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

The bridge IP, the key path, and every other connection detail belong in
that exercise's own `inventory.ini` and `ansible.cfg` — never here. The IP is
ephemeral (valid only for that container's lifetime), so an exercise's
inventory is valid only for the sitting that started its node. This file
describes the two shared images; it says nothing about any one exercise's
instance of them.

## Tear down

```bash
docker rm -f <exercise-dir>-node1 [<exercise-dir>-node2 ...]
ssh-keygen -R <node-bridge-ip>   # drop the stale host key from ~/.ssh/known_hosts
```

Every node is ephemeral: destroyed at the end of the sitting that created it,
regardless of whether the exercise concluded (the exercise's *files* persist;
the running container never needs to). `.tmp/ansible_node_key*` is gitignored
and regenerated per sitting like any other `.tmp/` scratch content.
