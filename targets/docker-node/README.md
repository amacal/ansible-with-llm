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

```bash
# once per sitting, if not already generated:
ssh-keygen -t ed25519 -f .tmp/ansible_node_key -N "" -q

docker build -t ansible-node-debian targets/docker-node/debian
docker run -d --name <exercise-slug>-node1 \
  -v "$(pwd)/.tmp/ansible_node_key.pub:/home/ansible/.ssh/authorized_keys:ro" \
  -p 0:22 ansible-node-debian
# or targets/docker-node/rhel for the RHEL-family image

docker port <exercise-slug>-node1 22   # find the mapped host port
```

The mapped port, the key path, and every other connection detail belong in
that exercise's own `inventory.ini` and `ansible.cfg` — never here. This file
describes the two shared images; it says nothing about any one exercise's
instance of them.

## Tear down

```bash
docker rm -f <exercise-slug>-node1 [<exercise-slug>-node2 ...]
```

Every node is ephemeral: destroyed at the end of the sitting that created it,
regardless of whether the exercise concluded (the exercise's *files* persist;
the running container never needs to). `.tmp/ansible_node_key*` is gitignored
and regenerated per sitting like any other `.tmp/` scratch content.
