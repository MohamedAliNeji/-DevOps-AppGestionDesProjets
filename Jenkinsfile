pipeline {
    agent any

    environment {
        DOCKERHUB_USER = 'mohamedalineji'
        BACKEND_IMAGE  = "${DOCKERHUB_USER}/gestion-backend:${BUILD_NUMBER}"
        FRONTEND_IMAGE = "${DOCKERHUB_USER}/gestion-frontend:${BUILD_NUMBER}"
    }

    tools {
        jdk 'JAVA_HOME'
        maven 'M2_HOME'
    }

    stages {

        stage('Git') {
            steps {
                checkout scm
            }
        }

        stage('Compilation') {
            steps {
                dir('backend') {
                    sh 'mvn clean compile'
                }
            }
        }

        stage('Construction') {
            steps {
                dir('backend') {
                    sh 'mvn package -DskipTests'
                }
            }
        }

        stage('Test') {
            steps {
                dir('backend') {
                    sh 'mvn test || true'
                }
            }
        }

        stage('Analyse Qualité - SonarQube') {
            steps {
                dir('backend') {
                    withSonarQubeEnv('SonarQube') {
                        sh 'mvn org.sonarsource.scanner.maven:sonar-maven-plugin:5.8.0.7211:sonar -Dsonar.projectKey=gestion-projets-backend'
                    }
                }
            }
        }

        stage('Création image + Push Registry') {
            steps {
                sh "docker build -t ${BACKEND_IMAGE} ./backend"
                sh "docker build -t ${FRONTEND_IMAGE} ./frontend"

                withCredentials([usernamePassword(credentialsId: 'dockerhub-creds', usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                    sh 'echo $DOCKER_PASS | docker login -u $DOCKER_USER --password-stdin'
                    sh "docker push ${BACKEND_IMAGE}"
                    sh "docker push ${FRONTEND_IMAGE}"
                }
            }
        }

        stage('Déploiement') {
            steps {
                sh 'docker compose down || true'
                sh 'docker compose up -d --build'
            }
        }
    }

    post {
        success {
            echo "Pipeline réussi. Images poussées : ${BACKEND_IMAGE}, ${FRONTEND_IMAGE}"
            echo "App accessible sur http://192.168.33.10:4200"
        }
        failure {
            echo 'Le pipeline a échoué — voir les logs ci-dessus.'
        }
    }
}
