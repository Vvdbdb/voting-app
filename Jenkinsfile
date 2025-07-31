pipeline {
    agent any

    environment {
        PROJECT_NAME = 'voting-app'
        REPO_URL = 'https://github.com/Vvdbdb/voting-app.git'
        REPO_BRANCH = 'master'
        COMPOSE_FILE = 'docker-compose.yml'
        SONAR_PROJECT_KEY = 'voting-app'
        SONAR_HOST_URL = 'http://sonarqube:9000'
        SONAR_TOKEN = credentials('sonarqube-token') 
    }

    stages {
        stage('Clone') {
            steps {
                echo "Clonage du dépôt ${REPO_URL}"
                git url: "${REPO_URL}", branch: "${REPO_BRANCH}"
                sh "ls -R ."
            }
        }

        stage('Build') {
            steps {
                echo "Construction des images Docker"
                sh "docker-compose -f ${COMPOSE_FILE} build db redis vote result worker"
            }
        }

        stage('Analyse SonarQube') {
        steps {
            sh '''
            docker compose run --rm sonarscanner \
                -Dsonar.projectKey=voting-app \
                -Dsonar.sources=vote,result,worker \
                -Dsonar.host.url=$SONAR_HOST_URL \
                -Dsonar.login=$SONAR_TOKEN \
                -Dsonar.scm.disabled=true
            '''
        }
        
        }
            post {
                always {
                    // Nettoyage du dossier temporaire après exécution
                    sh 'rm -rf sonar-src'
                }
            }
        }

        stage('Deploy') {
            steps {
                echo "Déploiement de l’application"
                sh "docker-compose -f ${COMPOSE_FILE} stop db redis vote result worker || true"
                sh "docker-compose -f ${COMPOSE_FILE} rm -f db redis vote result worker || true"
                sh "docker rm -f db redis vote result worker || true"
                sh "docker-compose -f ${COMPOSE_FILE} up -d db redis vote result worker"
            }
        }

        stage('Healthcheck') {
            steps {
                echo "Vérification des conteneurs"
                sh 'docker ps'
            }
        }
    }

    post {
        success {
            echo 'Pipeline terminé avec succès.'
            mail to: 'timotheekabore79@gmail.com',
                 subject: "✅ Pipeline Succès - ${PROJECT_NAME}",
                 body: "Le déploiement de ${PROJECT_NAME} a réussi avec succès."
        }
        failure {
            echo 'Pipeline échoué.'
            mail to: 'timotheekabore79@gmail.com',
                 subject: "❌ Pipeline Échoué - ${PROJECT_NAME}",
                 body: "Le déploiement de ${PROJECT_NAME} a échoué. Vérifiez Jenkins."
        }
    }
}
