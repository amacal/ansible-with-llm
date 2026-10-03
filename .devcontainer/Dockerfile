FROM mcr.microsoft.com/devcontainers/base:bookworm
SHELL ["/bin/bash", "-o", "pipefail", "-c"]

RUN apt-get update && apt-get install -y --no-install-recommends \
    ca-certificates curl git ripgrep python3 python3-venv pipx \
    openssh-client jq tzdata \
    qemu-kvm libvirt-daemon-system libvirt-clients virtinst \
    ruby-dev build-essential libvirt-dev libxml2-dev libxslt1-dev zlib1g-dev pkg-config \
    vagrant \
    && rm -rf /var/lib/apt/lists/* \
    && usermod -aG libvirt,kvm vscode

USER vscode
ENV PATH="/home/vscode/.local/bin:${PATH}"

RUN pipx install --include-deps ansible \
    && pipx inject ansible requests \
    && pipx install ansible-lint \
    && pipx inject ansible-lint ansible \
    && ansible --version

RUN vagrant plugin install vagrant-libvirt

RUN curl -fsSL https://claude.ai/install.sh | bash -s stable \
    && claude --version \
    && mkdir -p /home/vscode/.claude
