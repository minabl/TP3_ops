pipeline {
    agent any
    triggers {
        pollSCM('H/5 * * * *') // Vérifie les changements toutes les 5 minutes
    }
    environment {
        DOCKERHUB_CREDENTIALS = credentials('dockerhub') // Identifiants Docker Hub
        IMAGE_NAME_SERVER = 'minabf/mern-app-server' // Nom de l'image Docker pour le serveur
        IMAGE_NAME_CLIENT = 'minabf/mern-app-client' // Nom de l'image Docker pour le client
    }
    stages {
        stage('Checkout') { // Étape pour récupérer le code source
            steps {
                git branch: 'main',
                    url: 'git@github.com:minabl/TP3_ops.git', // URL du dépôt GitLab
                    credentialsId: 'github_id' // Identifiants SSH pour GitLab
            }
        }
        stage('Build Server Image') { // Construction de l'image Docker pour le serveur
            steps {
                dir('server') {
                    script {
                        dockerImageServer = docker.build("${IMAGE_NAME_SERVER}")
                    }
                }
            }
        }
        stage('Build Client Image') { // Construction de l'image Docker pour le client
            steps {
                dir('client') {
                    script {
                        dockerImageClient = docker.build("${IMAGE_NAME_CLIENT}")
                    }
                }
            }
        }
        stage('Scan Server Image') { // Analyse de l'image serveur avec Trivy
            steps {
                script {
                    sh """
                    docker run --rm -v /var/run/docker.sock:/var/run/docker.sock \\
                    aquasec/trivy:latest image --exit-code 0 \\
                    --severity LOW,MEDIUM,HIGH,CRITICAL \\
                    ${IMAGE_NAME_SERVER}
                    """
                }
            }
        }
        stage('Scan Client Image') { // Analyse de l'image client avec Trivy
            steps {
                script {
                    sh """
                    docker run --rm -v /var/run/docker.sock:/var/run/docker.sock \\
                    aquasec/trivy:latest image --exit-code 0 \\
                    --severity LOW,MEDIUM,HIGH,CRITICAL \\
                    ${IMAGE_NAME_CLIENT}
                    """
                }
            }
        }
        stage('Push Images to Docker Hub') { // Pousser les images sur Docker Hub
            steps {
                script {
                    docker.withRegistry('', "${DOCKERHUB_CREDENTIALS}") {
                        dockerImageServer.push()
                        dockerImageClient.push()
                    }
                }
            }
        }
    }
    post {
        always { // Nettoyage après chaque exécution, même en cas d'échec
            script {
                sh 'docker system prune -f' // Nettoie les artefacts Docker inutilisés
            }
        }
    }
}
