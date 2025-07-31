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

        // --- Étape d'analyse SonarQube intégrée ---
        stage('SonarQube Analysis') { // J'ai renommé l'étape pour être plus spécifique
            steps {
                script {
                    withSonarQubeEnv('SonarQube') { // 'SonarQube' est le nom de la configuration SonarQube dans Jenkins
                        withCredentials([string(credentialsId: 'sonarqube-token', variable: 'SONAR_TOKEN')]) {
                            echo "Lancement de l'analyse SonarQube pour le projet ${SONAR_PROJECT_KEY}"
                            // La commande docker run pour le SonarScanner.
                            // Le nom du réseau est 'voting-app_private_net' car ton dossier de projet est probablement 'voting-app'.
                            sh '''
                                docker run --rm \\
                                  --network voting-app_private_net \\
                                  -e SONAR_HOST_URL=$SONAR_HOST_URL \\
                                  -e SONAR_TOKEN=$SONAR_TOKEN \\
                                  -v $PWD:/usr/src \\
                                  sonarsource/sonar-scanner-cli:latest \\
                                  sonar-scanner \\
                                    -Dsonar.projectKey=$SONAR_PROJECT_KEY \\
                                    -Dsonar.sources=./vote,./result,./worker \\
                                    -Dsonar.host.url=$SONAR_HOST_URL
                            '''
                        }
                    }
                }
            }
        }
        // --- Fin de l'étape SonarQube ---

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
