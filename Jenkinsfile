pipeline {

    agent any

    environment {
        APP_NAME = 'online-banking'
        DOCKER_IMAGE = 'xvishu/online-banking'
        MAVEN_IMAGE = 'maven:3.9-eclipse-temurin-21'
    }

    stages {

        stage('Initialize') {
            steps {
                echo 'Initializing Jenkins CI Pipeline'

                sh '''
                    echo "Workspace: ${WORKSPACE}"
                    echo "Build Number: ${BUILD_NUMBER}"
                    docker --version
                '''
            }
        }

        stage('Checkout') {
            steps {
                echo 'Checking out source code...'
                checkout scm
            }
        }

        stage('Build & Test') {
            steps {
                echo 'Running Maven build and tests...'

                sh '''
                    docker run --rm \
                      --volumes-from nexaflow-jenkins \
                      -w "${WORKSPACE}" \
                      ${MAVEN_IMAGE} \
                      mvn clean package
                '''
            }
        }

        stage('Build Docker Image') {
            steps {
                echo 'Building Docker image...'

                sh '''
                    docker build \
                      -f Dockerfile.devops \
                      -t ${DOCKER_IMAGE}:${BUILD_NUMBER} \
                      -t ${DOCKER_IMAGE}:latest \
                      .
                '''
            }
        }

        stage('Docker Login') {
            steps {
                echo 'Logging in to Docker Hub...'

                withCredentials([
                    usernamePassword(
                        credentialsId: '552968b7-7c07-451e-9d28-76e59c51329a',
                        usernameVariable: 'DOCKER_USERNAME',
                        passwordVariable: 'DOCKER_PASSWORD'
                    )
                ]) {
                    sh '''
                        echo "$DOCKER_PASSWORD" | \
                        docker login \
                        --username "$DOCKER_USERNAME" \
                        --password-stdin
                    '''
                }
            }
        }

        stage('Push Docker Image') {
            steps {
                echo 'Pushing Docker image to Docker Hub...'

                sh '''
                    docker push ${DOCKER_IMAGE}:${BUILD_NUMBER}
                    docker push ${DOCKER_IMAGE}:latest
                '''
            }
        }

    }

    post {

        success {
            echo """
=========================================
 CI PIPELINE SUCCESSFUL
=========================================
Application build : SUCCESS
Docker build      : SUCCESS
Docker push       : SUCCESS

Image:
${DOCKER_IMAGE}:${BUILD_NUMBER}
${DOCKER_IMAGE}:latest
=========================================
"""
        }

        failure {
            echo '''
=========================================
 CI PIPELINE FAILED
=========================================
Check the Jenkins console output.
=========================================
'''
        }

        always {
            echo "Jenkins Build #${BUILD_NUMBER} completed."
        }
    }
}