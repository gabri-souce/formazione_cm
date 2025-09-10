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
    apt-get install -y \
    docker-ce-cli \
    python3-pip \
    && \
    # Install Ansible using pip with break-system-packages flag
    pip3 install ansible --break-system-packages && \
    # Create docker group and add jenkins user to it
    groupadd docker && \
    usermod -aG docker jenkins && \
    apt-get clean && \
    rm -rf /var/lib/apt/lists/*

USER jenkins

# Copy Jenkins configuration
COPY executors.groovy /usr/share/jenkins/ref/init.groovy.d/executors.groovy
COPY ansible.cfg /etc/ansible/ansible.cfg

EXPOSE 8080 50000