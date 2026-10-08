FROM ubuntu:22.04

ENV DEBIAN_FRONTEND=noninteractive

RUN apt-get update && apt-get install -y \
    openssh-server \
    sudo \
    python3 \
    curl \
    && rm -rf /var/lib/apt/lists/*

# Create deploy user
RUN useradd -m -s /bin/bash deploy \
    && echo "deploy:deploy" | chpasswd \
    && usermod -aG sudo deploy \
    && echo "deploy ALL=(ALL) NOPASSWD:ALL" >> /etc/sudoers

# SSH setup
RUN mkdir -p /var/run/sshd
COPY ../ssh/id_lab.pub /tmp/id_lab.pub
RUN mkdir -p /home/deploy/.ssh \
    && cat /tmp/id_lab.pub  > /home/deploy/.ssh/authorized_keys \
    && chown -R deploy:deploy /home/deploy/.ssh \
    && chmod 700 /home/deploy/.ssh \
    && chmod 600 /home/deploy/.ssh/authorized_keys \
    && rm /tmp/id_lab.pub

EXPOSE 22

CMD ["/usr/sbin/sshd", "-D"]
