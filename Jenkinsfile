pipeline {
    agent any

    environment {
        DOCKER_CREDENTIALS = credentials('dockerhub-credentials')  // Use your DockerHub credentials
        AWS_CREDENTIALS = credentials('aws-credentials')            // Use your AWS credentials
        KUBE_CONFIG = credentials('kube-config')                    // If needed for kubectl commands
    }

    stages {
        stage('Checkout') {
            steps {
                git branch: 'main', url: 'git@github.com:muhammadwaqas964/devsecops-nodejs-app.git'
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    sh 'docker build -t my-nodejs-app .'
                }
            }
        }

        stage('Push to DockerHub') {
            steps {
                script {
                    docker.withRegistry('https://index.docker.io/v1/', DOCKER_CREDENTIALS) {
                        sh 'docker push muhammadwaqas366/my-nodejs-app'
                    }
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                script {
                    sh 'kubectl apply -f deployment.yaml'
                }
            }
        }
    }

    post {
        always {
            echo 'Cleaning up Docker images...'
            sh 'docker rmi my-nodejs-app || true'
        }
    }
}

