pipeline {
    agent any

    tools {
        maven 'Maven3'
    }

    environment {
        PATH = "/opt/homebrew/bin:/usr/local/bin:${env.PATH}"
//        JAVA_HOME = "/usr/local/opt/openjdk@17/bin/java"
        SONARQUBE_SERVER = "SonarQubeServer"
        SONAR_TOKEN = credentials("sonarcloud-token")

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

        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv('SonarCloud') {
                    withCredentials([string(credentialsId: 'sonarcloud-token', variable: 'SONAR_TOKEN')]) {
                        sh """
                            ${tool 'SonarScanner'}/bin/sonar-scanner \
                            -Dsonar.projectKey=SEP2_week5_assignment \
                            -Dsonar.sources=src \
                            -Dsonar.projectName=SEP2_week5_assignment \
                            -Dsonar.host.url=https://sonarcloud.io/ \
                            -Dsonar.login=${SONAR_TOKEN} \
                            -Dsonar.java.binaries=${WORKSPACE}/target/classes
                        """
                    }
                }
            }
        }

//        stage('SonarCloud Analysis') {
//            steps {
//                withSonarQubeEnv('SonarCloud') {
//                    script {
//                        def scannerHome = tool 'SonarScanner'
//                        sh "${scannerHome}/bin/sonar-scanner"
//                    }
//                }
//            }
//        }

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