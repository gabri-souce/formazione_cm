FROM ubuntu:20.04

# Set non-interactive frontend and environment variables
ENV DEBIAN_FRONTEND=noninteractive
ENV SSH_USER=ansible-user
ARG SSH_USER_PASSWORD

# Install required packages
RUN apt-get update && \
    apt-get install -y --no-install-recommends \
    openssh-server \
    sudo \
    curl \
    apt-transport-https \
    ca-certificates \
    gnupg \
    lsb-release \
    && rm -rf /var/lib/apt/lists/*

# Create user with password from vault
RUN useradd -m -s /bin/bash $SSH_USER && \
    echo "$SSH_USER ALL=(ALL) NOPASSWD:ALL" >> /etc/sudoers && \
    echo "$SSH_USER:$SSH_USER_PASSWORD" | chpasswd

# Setup SSH
RUN mkdir -p /home/$SSH_USER/.ssh && \
    chmod 700 /home/$SSH_USER/.ssh

# Copy SSH public key
COPY authorized_keys /home/$SSH_USER/.ssh/authorized_keys
RUN chown -R $SSH_USER:$SSH_USER /home/$SSH_USER/.ssh && \
    chmod 600 /home/$SSH_USER/.ssh/authorized_keys

# Configure SSH server
RUN mkdir -p /var/run/sshd && \
    sed -i 's/#PermitRootLogin prohibit-password/PermitRootLogin no/' /etc/ssh/sshd_config && \
    sed -i 's/#PasswordAuthentication yes/PasswordAuthentication no/' /etc/ssh/sshd_config && \
    sed -i 's/#PubkeyAuthentication yes/PubkeyAuthentication yes/' /etc/ssh/sshd_config

# Install Docker Engine using official Ubuntu packages (FIXED)
RUN install -m 0755 -d /etc/apt/keyrings && \
    curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc && \
    chmod a+r /etc/apt/keyrings/docker.asc && \
    echo "deb [arch=amd64 signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/ubuntu $(. /etc/os-release && echo \"$VERSION_CODENAME\") stable" | tee /etc/apt/sources.list.d/docker.list > /dev/null && \
    apt-get update && \
    apt-get install -y --no-install-recommends \
    docker-ce \
    docker-ce-cli \
    containerd.io \
    docker-buildx-plugin \
    docker-compose-plugin && \
    groupadd -f docker && \
    usermod -aG docker $SSH_USER && \
    rm -rf /var/lib/apt/lists/*

# Clean up
RUN apt-get clean && \
    rm -rf /var/lib/apt/lists/* /tmp/* /var/tmp/*

# Expose SSH port
EXPOSE 22

# Health check
HEALTHCHECK --interval=30s --timeout=10s --start-period=5s --retries=3 \
    CMD netstat -an | grep :22 > /dev/null || exit 1

# Start SSH service
CMD ["/usr/sbin/sshd", "-D", "-e"]