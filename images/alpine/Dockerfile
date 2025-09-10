FROM alpine:latest

# Set user password from environment
ENV SSH_USER=ansible-user
ENV SSH_USER_PASSWORD=${SSH_USER_PASSWORD}

RUN apk update && \
    apk add --no-cache openssh-server openssh-client sudo && \
    rm -rf /var/cache/apk/*

RUN adduser -D -s /bin/bash $SSH_USER && \
    echo "$SSH_USER ALL=(ALL) NOPASSWD:ALL" >> /etc/sudoers && \
    echo "$SSH_USER:$SSH_USER_PASSWORD" | chpasswd

RUN mkdir -p /home/$SSH_USER/.ssh && \
    chmod 700 /home/$SSH_USER/.ssh

COPY authorized_keys /home/$SSH_USER/.ssh/authorized_keys
RUN chown -R $SSH_USER:$SSH_USER /home/$SSH_USER/.ssh && \
    chmod 600 /home/$SSH_USER/.ssh/authorized_keys

RUN ssh-keygen -A && \
    sed -i 's/#PermitRootLogin prohibit-password/PermitRootLogin no/' /etc/ssh/sshd_config && \
    sed -i 's/#PasswordAuthentication yes/PasswordAuthentication no/' /etc/ssh/sshd_config

EXPOSE 22

CMD ["/usr/sbin/sshd", "-D"]