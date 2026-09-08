pipeline {

    agent any

    environment {
        APP_NAME = 'online-banking'
        DOCKER_IMAGE = 'online-banking'
        MAVEN_IMAGE = 'maven:3.9-eclipse-temurin-21'
    }

    stages {

        stage('Initialize') {
            steps {
                echo '========================================='
                echo 'Initializing Jenkins CI Pipeline'
                echo '========================================='

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

        stage('Docker Image Test') {
            steps {
                echo 'Checking Docker image...'

                sh '''
                    docker images ${DOCKER_IMAGE}
                '''
            }
        }

    }

    post {

        success {
            echo '''
=========================================
 CI PIPELINE SUCCESSFUL
=========================================
Application build: SUCCESS
Docker build:      SUCCESS
=========================================
'''
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