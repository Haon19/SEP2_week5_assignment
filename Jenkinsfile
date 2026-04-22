import java.nio.file.Path

pipeline {
    agent any

    tools {
        maven 'Maven3'
    }

    environment {
        Path = ""
        JAVA_HOME = ""
        SONARQUBE_SERVER = ""
        SONAR_TOKEN = credentials("")

        DOCKERHUB_CREDENTIALS_ID = 'Docker_Hub'
        DOCKERHUB_REPO = 'haon19/sonarqube-pipeline'
        DOCKER_IMAGE_TAG = 'latest'
    }

    stages {

        stage('Checkout') {
            steps {
                git branch: 'main',
                        url: 'https://github.com/Haon19/SEP2_week5_assignment.git'
            }
        }

        stage('Build') {
            steps {
                sh 'mvn clean install'
            }
        }

        stage('SonarCloud Analysis') {
            steps {
                withSonarQubeEnv('SonarQubeServer') {
                    sh "${tool 'SonarScanner'}/bin/sonar-scanner"//diff maybe
                }
            }
        }

        stage('Quality Gate') {
            steps {
                timeout(time: 5, unit: 'MINUTES') {
                    waitForQualityGate abortPipeline: true
                }
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    docker.build("${DOCKERHUB_REPO}:${DOCKER_IMAGE_TAG}")
                }
            }
        }

        stage('Push Docker Image') {
            steps {
                withCredentials([usernamePassword(
                        credentialsId: "${DOCKERHUB_CREDENTIALS_ID}",
                        usernameVariable: 'DOCKER_USER',
                        passwordVariable: 'DOCKER_PASS'
                )]) {
                    sh '''
                 echo $DOCKER_PASS | docker login -u $DOCKER_USER --password-stdin
                 docker push ${DOCKERHUB_REPO}:${DOCKER_IMAGE_TAG}
             '''
                }
            }
        }
    }
}