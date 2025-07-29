pipeline {
    agent any

    environment {
        PROJECT_NAME = 'voting-app'
        REPO_URL = 'https://github.com/Vvdbdb/voting-app.git'
        REPO_BRANCH = 'master'
        COMPOSE_FILE = "docker-compose.yml"
        SONAR_SCANNER = "SonarScanner" // nom défini dans Jenkins
        SONAR_PROJECT_KEY = "voting-app" // même nom que sur l'interface SonarQube
        SONAR_HOST_URL = "http://sonarqube:9000" // correspond au nom du service docker
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
                sh "docker-compose -f ${COMPOSE_FILE} build db redis vote result worker"
            }
        }

        stage('Check Scanner') {
            steps {
                echo "Vérification du scanner SonarQube"
                sh "echo Scanner path: ${SONAR_SCANNER}"
                sh "ls -l ${SONAR_SCANNER}/bin/"
                sh "${SONAR_SCANNER}/bin/sonar-scanner --version"
            }
        }

        stage('SonarQube') {
            steps {
                echo "Analyse SonarQube en cours..."
                withSonarQubeEnv('SonarQube') {
                    withCredentials([string(credentialsId: 'sonarqube-token', variable: 'SONAR_TOKEN')]) {
                        sh "${SONAR_SCANNER}/bin/sonar-scanner \
                            -Dsonar.projectKey=${SONAR_PROJECT_KEY} \
                            -Dsonar.sources=. \
                            -Dsonar.host.url=${SONAR_HOST_URL} \
                            -Dsonar.login=$SONAR_TOKEN"
                    }
                }
            }
        }

        stage('Deploy') {
            steps {
                echo "Déploiement de l’application"
                sh "docker-compose -f ${COMPOSE_FILE} stop db redis vote result worker || true"
                sh "docker-compose -f ${COMPOSE_FILE} rm -f db redis vote result worker || true"
    
                sh "docker rm -f db redis vote result worker || true"
        
                echo "Démarrage des services applicatifs"
                sh "docker-compose -f ${COMPOSE_FILE} up -d db redis vote result worker"
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
            mail to: 'timotheekabore79@gmail.com',
                subject: "Pipeline succès - ${PROJECT_NAME}",
                body: "Le déploiement de ${PROJECT_NAME} a réussi !"
        }
        failure {
            echo 'Pipeline échoué.'
            mail to: 'timotheekabore79@gmail.com',
                subject: "Pipeline échoué - ${PROJECT_NAME}",
                body: "Le déploiement de ${PROJECT_NAME} a échoué."
        }
    }
}
