FROM jenkins/jenkins:lts

USER root

# Installer Docker dans le conteneur Jenkins
RUN apt-get update && \
    apt-get install -y docker.io && \
    usermod -aG docker jenkins && \
    apt-get clean

# Assurez-vous que Jenkins a accès au socket Docker
#RUN chown jenkins:docker /var/run/docker.sock

USER jenkins
