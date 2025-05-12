# Dockerfile for Alpine Linux (latest) with Supervisor, SSHD, and Puppet Agent

# Use Alpine latest as the base image
FROM alpine:latest

LABEL maintainer="Your Name <youremail@example.com>"
LABEL description="Alpine Linux with Supervisor to run SSHD and Puppet Agent."

# Environment variables
ENV PUPPET_MAJOR_VERSION="7" 
ENV SSH_PORT="22"
# WARNING: Setting a default root password in a Dockerfile is insecure.
# For production, use SSH keys and remove password authentication or inject secrets.
ENV ROOT_PASSWORD="12345678"

# Install necessary packages and Puppet Agent
RUN apk update && \
    # Install prerequisites
    apk add --no-cache \
        bash \
        ca-certificates \
        wget \
        supervisor \
        openssh \
        shadow \
        sudo 
    # Add Puppet GPG key
#RUN    wget -O /etc/apk/keys/puppet.gpg.pub https://yum.puppet.com/PUPPET-GPG-KEY-2025 && \
#       # Add Puppet repository for the specified major version
#       # Note: Check Puppet's official documentation for the most current repository URL for Alpine
#       echo "https://apk.puppet.com/alpine/v${PUPPET_MAJOR_VERSION}/main" >> /etc/apk/repositories && \
#       # Update apk cache again after adding new repository
#       apk update && \
#       # Install Puppet agent
#       apk add --no-cache puppet-agent && \
#       # Clean up apk cache
#       rm -rf /var/cache/apk/*

# Configure SSHD
RUN echo "root:${ROOT_PASSWORD}" | chpasswd && \
    # Permit root login with password (INSECURE for production)
    sed -i 's/^#?PermitRootLogin prohibit-password/PermitRootLogin yes/' /etc/ssh/sshd_config && \
    sed -i 's/^#?PasswordAuthentication yes/PasswordAuthentication yes/' /etc/ssh/sshd_config && \
    sed -i 's/^#?ChallengeResponseAuthentication yes/ChallengeResponseAuthentication no/' /etc/ssh/sshd_config && \
    # Generate SSH host keys if they don't exist (sshd will also do this on first start if missing)
    ssh-keygen -A && \
    # Create a directory for sshd to run
    mkdir -p /run/sshd && \
    chmod 0755 /run/sshd

# Configure Supervisor
# Copy the supervisord.conf file into the container
COPY supervisord.conf /etc/supervisor/conf.d/supervisord.conf

# Create log directory for supervisor program logs
RUN mkdir -p /var/log/supervisor

# Expose SSH port
EXPOSE ${SSH_PORT}

# Set the default command to run Supervisor
CMD ["/usr/bin/supervisord", "-c", "/etc/supervisor/conf.d/supervisord.conf"]

