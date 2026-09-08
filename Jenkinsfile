pipeline {

    agent any

    environment {
        DOCKER_IMAGE = "xvishu/online-banking"
    }

    stages {

        stage('Init') {
            steps {
                echo "========================================"
                echo "Online Banking CI Pipeline"
                echo "Build Number: ${BUILD_NUMBER}"
                echo "========================================"
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
                    -w "$WORKSPACE" \
                    maven:3.9-eclipse-temurin-21 \
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
                        printf '%s' "$DOCKER_PASSWORD" | \
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
            echo "========================================"
            echo "CI PIPELINE SUCCESS"
            echo "Docker Image: ${DOCKER_IMAGE}:${BUILD_NUMBER}"
            echo "========================================"
        }

        failure {
            echo "========================================"
            echo "CI PIPELINE FAILED"
            echo "Check the Jenkins console output."
            echo "========================================"
        }

        always {
            echo "Jenkins Build #${BUILD_NUMBER} completed."
        }
    }
}