pipeline {
    agent any

    environment {
        IMAGE_NAME = 'basamr/bookstore'
        IMAGE_TAG  = "${env.BUILD_NUMBER}"
        PATH = "C:\Program Files\Git\cmd\git.exe"
        KUBECONFIG = 'C:\\Users\\b.kamel\\.kube\\config'
    }
    tools {
        maven 'maven3.9' // This must match the Name you gave in Global Tool Configuration
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Build JAR') {
            steps {
                bat 'mvn -DskipTests clean package'
            }
        }

        stage('Build Docker Image') {
            steps {
                bat "docker build -t %IMAGE_NAME%:%IMAGE_TAG% ."
            }
        }

        stage('Push Docker Image') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'dockerhub-pat',
                    usernameVariable: 'DOCKER_USER',
                    passwordVariable: 'DOCKER_PASS'
                )]) {
                    bat '''
                    echo %DOCKER_PASS% | docker login -u %DOCKER_USER% --password-stdin
                    docker push %IMAGE_NAME%:%IMAGE_TAG%
                    '''
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                bat '''
                kubectl apply -f k8s
                kubectl set image deployment/bookstore-app bookstore-app=%IMAGE_NAME%:%IMAGE_TAG%
                kubectl rollout status deployment/bookstore-app
                '''
            }
        }
    }

    post {
        success {
            echo 'Deployment completed successfully.'
        }
        failure {
            echo 'Pipeline failed.'
        }
    }
}
