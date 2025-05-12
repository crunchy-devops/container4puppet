# Dockerfile for Ubuntu Latest with Supervisor, SSHD, and Puppet Agent

# Use Ubuntu latest as the base image
FROM ubuntu:latest

LABEL maintainer="Your Name <youremail@example.com>"
LABEL description="Ubuntu Latest with Supervisor to run SSHD and Puppet Agent."

# Environment variables
ENV PUPPET_AGENT_VERSION="7"
ENV SSH_PORT="22"
# WARNING: Setting a default root password in a Dockerfile is insecure.
# For production, use SSH keys and remove password authentication or inject secrets.
ENV ROOT_PASSWORD="12345678"
ENV DEBIAN_FRONTEND=noninteractive

# Install necessary packages
RUN apt-get update && \
    apt-get install -y --no-install-recommends \
        wget \
        ca-certificates \
        gnupg \
        supervisor \
        openssh-server \
        openssh-client \
        passwd \
        sudo \
        lsb-release \
        which && \
    # Add Puppet repository
    wget https://apt.puppet.com/puppet${PUPPET_AGENT_VERSION}-release-$(lsb_release -cs).deb && \
    dpkg -i puppet${PUPPET_AGENT_VERSION}-release-$(lsb_release -cs).deb && \
    apt-get update && \
    # Install Puppet agent
    apt-get install -y --no-install-recommends puppet-agent && \
    # Clean up
    rm puppet${PUPPET_AGENT_VERSION}-release-$(lsb_release -cs).deb && \
    apt-get clean && \
    rm -rf /var/lib/apt/lists/*

# Configure SSHD
RUN echo "root:${ROOT_PASSWORD}" | chpasswd && \
    # Permit root login with password (INSECURE for production)
    sed -i 's/^#?PermitRootLogin prohibit-password/PermitRootLogin yes/' /etc/ssh/sshd_config && \
    sed -i 's/^#?PasswordAuthentication yes/PasswordAuthentication yes/' /etc/ssh/sshd_config && \
    sed -i 's/^#?ChallengeResponseAuthentication yes/ChallengeResponseAuthentication no/' /etc/ssh/sshd_config && \
    # Ensure UsePAM is yes for sshd to work correctly with user sessions
    sed -i 's/^#?UsePAM no/UsePAM yes/' /etc/ssh/sshd_config && \
    # Generate SSH host keys.
    ssh-keygen -A && \
    # Create a directory for sshd to run, OpenSSH on Debian/Ubuntu expects /run/sshd
    mkdir -p /run/sshd && \
    chmod 0755 /run/sshd

# Configure Supervisor
# Copy the supervisord.conf file into the container
# Ensure you have supervisord.conf from a previous example in the build context
COPY supervisord.conf /etc/supervisor/conf.d/supervisord.conf

# Create log directory for supervisor program logs
RUN mkdir -p /var/log/supervisor

# Expose SSH port
EXPOSE ${SSH_PORT}

# Set the default command to run Supervisor
CMD ["/usr/bin/supervisord", "-n", "-c", "/etc/supervisor/conf.d/supervisord.conf"]

