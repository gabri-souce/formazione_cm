FROM jenkins/jenkins:lts

USER root

# Install Docker, Ansible, and required tools
RUN apt-get update && \
    apt-get install -y \
    software-properties-common \
    apt-transport-https \
    ca-certificates \
    curl \
    gnupg \
    lsb-release && \
    curl -fsSL https://download.docker.com/linux/debian/gpg | gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg && \
    echo "deb [arch=amd64 signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/debian $(lsb_release -cs) stable" | tee /etc/apt/sources.list.d/docker.list > /dev/null && \
    apt-get update && \
    apt-get install -y docker-ce-cli ansible && \
    apt-get clean && \
    rm -rf /var/lib/apt/lists/*

# Fix Docker group (gestisce correttamente GID già esistente)
RUN groupmod -n docker $(getent group 999 | cut -d: -f1) && \
    usermod -aG docker jenkins

# Installa plugin Jenkins con jenkins-plugin-cli
RUN jenkins-plugin-cli --plugins \
    docker-plugin \
    ansible \
    git \
    ssh-slaves \
    workflow-aggregator \
    credentials-binding \
    blueocean

# Copy Jenkins configuration
COPY executors.groovy /usr/share/jenkins/ref/init.groovy.d/executors.groovy
COPY ansible.cfg /etc/ansible/ansible.cfg

# Set environment variables
ENV DOCKER_HOST=unix:///var/run/docker.sock

USER jenkins

EXPOSE 8080 50000


