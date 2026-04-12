pipeline {
    agent any

    environment {
        IMAGE_NAME = 'basamr/bookstore'
        IMAGE_TAG  = "${env.BUILD_NUMBER}"
        KUBECONFIG = 'C:\\Users\\b.kamel\\.kube\\config'
        PATH = "C:\Program Files\Git\cmd\git.exe;C:\\Program Files\\Rancher Desktop\\resources\\resources\\win32\\bin"
    }

    tools {
        maven 'maven3.9'
    }

    stages {
        stage('Checkout App Repo') {
            steps {
                deleteDir()
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

        stage('Checkout K8s Repo') {
            steps {
                dir('k8s-manifests') {
                    deleteDir()
                    git branch: 'main', url: 'https://github.com/basselamr/k8s.git'
                }
            }
        }

        stage('Kube Debug') {
    steps {
        bat 'echo %KUBECONFIG%'
        bat 'kubectl config current-context'
        bat 'kubectl config get-contexts'
        bat 'kubectl cluster-info'
    }
}

       stage('Deploy to Kubernetes') {
            steps {
                bat '''
                kubectl apply -f k8s-manifests
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
