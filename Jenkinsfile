pipeline {
    agent any

    environment {
        PROJECT_NAME = 'voting-app'
        REPO_URL = 'https://github.com/Vvdbdb/voting-app.git'
        REPO_BRANCH = 'main'
        COMPOSE_FILE = "docker-compose.yml"
    }

    stages {
        stage('Clone') {
            steps {
                echo "Clonage du dépôt ${REPO_URL}"
                git url: "${REPO_URL}", branch: "${REPO_BRANCH}"
            }
        }

        stage('Build') {
            steps {
                echo "Construction des images Docker"
                sh "docker compose -f ${COMPOSE_FILE} build"
            }
        }

        stage('Deploy') {
            steps {
                echo "Déploiement de l’application"
                sh "docker compose -f ${COMPOSE_FILE} down -v"
                sh "docker compose -f ${COMPOSE_FILE} up -d"
            }
        }

        stage('Healthcheck') {
            steps {
                echo "Vérification de l’état des conteneurs"
                sh 'docker ps'
            }
        }
    }

    post {
        success {
            echo 'Pipeline terminé avec succès.'
            mail to: 'timopoubelle@gmail.com',
                subject: "Pipeline succès - ${PROJECT_NAME}",
                body: "Le déploiement de ${PROJECT_NAME} a réussi !"
        }
        failure {
            echo 'Pipeline échoué.'
            mail to: 'timopoubelle@gmail.com',
                subject: "Pipeline échoué - ${PROJECT_NAME}",
                body: "Le déploiement de ${PROJECT_NAME} a échoué."
        }
    }
}
