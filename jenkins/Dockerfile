FROM jenkins/jenkins:lts

USER root

# Installer Docker et docker-compose dans l'image Jenkins
RUN apt-get update && \
    apt-get install -y docker.io curl python3-pip && \
    pip3 install docker-compose && \
    usermod -aG docker jenkins && \
    apt-get clean && \
    rm -rf /var/lib/apt/lists/*

USER jenkins
