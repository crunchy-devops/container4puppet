# Dockerfile for CentOS Stream 9 (latest) with Supervisor, SSHD, and Puppet Agent

# Use CentOS Stream 9 as the base image
# Note: 'centos:latest' currently points to CentOS Stream 9.
# If you need CentOS 7, use 'centos:7'. This Dockerfile targets CentOS Stream 9.
FROM centos

LABEL maintainer="Your Name <youremail@example.com>"
LABEL description="CentOS Stream 9 with Supervisor to run SSHD and Puppet Agent."

# Environment variables
ENV PUPPET_AGENT_VERSION="7" 
ENV SSH_PORT="22"
# WARNING: Setting a default root password in a Dockerfile is insecure.
# For production, use SSH keys and remove password authentication or inject secrets.
ENV ROOT_PASSWORD="12345678"

# Install necessary packages
# CentOS Stream 9 uses dnf.
RUN dnf update -y && \
    # Install EPEL repository which contains supervisor
    dnf install -y epel-release && \
    # Install Puppet repository
    rpm -Uvh https://yum.puppet.com/puppet${PUPPET_AGENT_VERSION}-release-el-9.noarch.rpm && \
    # Install Supervisor, OpenSSH server & client, Puppet agent, and other utilities
    dnf install -y \
        supervisor \
        openssh-server \
        openssh-clients \
        passwd \
        sudo \
        which \
        initscripts \
        puppet-agent && \
    dnf clean all && \
    rm -rf /var/cache/dnf/*

# Configure SSHD
RUN echo "root:${ROOT_PASSWORD}" | chpasswd && \
    # Permit root login with password (INSECURE for production)
    sed -i 's/^#?PermitRootLogin prohibit-password/PermitRootLogin yes/' /etc/ssh/sshd_config && \
    sed -i 's/^#?PasswordAuthentication yes/PasswordAuthentication yes/' /etc/ssh/sshd_config && \
    sed -i 's/^#?ChallengeResponseAuthentication yes/ChallengeResponseAuthentication no/' /etc/ssh/sshd_config && \
    # Generate SSH host keys.
    # sshd service unit in systemd-based images often handles this.
    # For supervisor, we ensure they are present or generated by sshd on its first start.
    ssh-keygen -A && \
    # Create a directory for sshd to run
    mkdir -p /run/sshd && \
    chmod 0755 /run/sshd

# Configure Supervisor
# Copy the supervisord.conf file into the container
# Ensure you have supervisord.conf from the previous Alpine example in the build context
COPY supervisord.conf /etc/supervisor/conf.d/supervisord.conf

# Create log directory for supervisor program logs
RUN mkdir -p /var/log/supervisor

# Expose SSH port
EXPOSE ${SSH_PORT}

# Set the default command to run Supervisor
CMD ["/usr/bin/supervisord", "-c", "/etc/supervisor/conf.d/supervisord.conf"]

