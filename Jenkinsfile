pipeline {
    agent any

    environment {
        SONARQUBE_SERVER = 'SonarQubeServer'  // The name of the SonarQube server configured in Jenkins
        SONAR_TOKEN = credentials('SonarQube_Key') // Store the token securely
        DOCKERHUB_CREDENTIALS_ID = 'Docker_Hub'
        DOCKERHUB_REPO = 'trungv12/sonarqubejenkins'
        DOCKER_IMAGE_TAG = 'latest_v1'
    }

    stages {
        stage('Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/travisv12/SonarQube_Jenkins_Setup.git'
            }
        }

        stage('Build') {
            steps {
                bat 'mvn clean install'
            }
        }

        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv('SonarQubeServer') {
                    bat """
                        sonar-scanner ^
                        -Dsonar.projectKey=devops-demo ^
                        -Dsonar.sources=src ^
                        -Dsonar.projectName=DevOps-Demo ^
                        -Dsonar.host.url=http://localhost:9000 ^
                        -Dsonar.login=${env.SONAR_TOKEN} ^
                        -Dsonar.java.binaries=target/classes
                    """
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
                stage('Push Docker Image to Docker Hub') {
                    steps {
                        script {
                            withCredentials([usernamePassword(credentialsId: DOCKERHUB_CREDENTIALS_ID, usernameVariable: 'DOCKERHUB_USERNAME', passwordVariable: 'DOCKERHUB_PASSWORD')]) {
                                bat """
                                echo %DOCKERHUB_PASSWORD% | docker login -u %DOCKERHUB_USERNAME% --password-stdin
                                docker tag ${DOCKERHUB_REPO}:${DOCKER_IMAGE_TAG} ${DOCKERHUB_REPO}:${DOCKER_IMAGE_TAG}
                                docker push ${DOCKERHUB_REPO}:${DOCKER_IMAGE_TAG}
                                """
                            }
                        }
                    }
                }

    }
}
