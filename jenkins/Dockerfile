FROM jenkins/jenkins:lts

USER root

# Installer Docker dans le conteneur Jenkins
RUN apt-get update && \
    apt-get install -y docker.io && \
    usermod -aG docker jenkins && \
    apt-get clean

USER jenkins
