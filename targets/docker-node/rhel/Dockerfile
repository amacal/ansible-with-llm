FROM rockylinux:9

RUN dnf install -y openssh-server python3 sudo \
    && dnf clean all \
    && ssh-keygen -A \
    && useradd -m -s /bin/bash ansible \
    && echo 'ansible ALL=(ALL) NOPASSWD:ALL' > /etc/sudoers.d/ansible \
    && chmod 440 /etc/sudoers.d/ansible \
    && mkdir -p /home/ansible/.ssh \
    && chown -R ansible:ansible /home/ansible/.ssh \
    && chmod 700 /home/ansible/.ssh

EXPOSE 22
CMD chmod 600 /home/ansible/.ssh/authorized_keys 2>/dev/null; exec /usr/sbin/sshd -D
