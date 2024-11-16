pipeline {
    agent any

    environment {
        DOCKER_IMAGE_NAME = "muhammadwaqas366/devsecops-nodejs-app"
        DOCKER_IMAGE_TAG = "latest"
        DOCKER_CREDENTIALS = "dockerhub-credentials"
        GITHUB_CREDENTIALS = "Github_AWS_PEOJECT"
        K8S_NAMESPACE = "default"
        KUBECONFIG_PATH = "/home/jenkins/.kube/config"
        AWS_CREDENTIALS = "aws-credentials"
    }

    stages {
        stage('Checkout SCM') {
            steps {
                checkout scm
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    withCredentials([usernamePassword(credentialsId: DOCKER_CREDENTIALS, passwordVariable: 'DOCKER_PASSWORD', usernameVariable: 'DOCKER_USERNAME')]) {
                        sh 'echo $DOCKER_PASSWORD | docker login --username $DOCKER_USERNAME --password-stdin'
                    }
                    sh 'docker build -t $DOCKER_IMAGE_NAME:$DOCKER_IMAGE_TAG .'
                }
            }
        }

        stage('Security Scan') {
            steps {
                script {
                    echo "Running security scan on Docker image"
                    // Example: sh 'trivy image $DOCKER_IMAGE_NAME:$DOCKER_IMAGE_TAG'
                }
            }
        }

        stage('Push Image to DockerHub') {
            steps {
                script {
                    withCredentials([usernamePassword(credentialsId: DOCKER_CREDENTIALS, passwordVariable: 'DOCKER_PASSWORD', usernameVariable: 'DOCKER_USERNAME')]) {
                        sh 'docker push $DOCKER_IMAGE_NAME:$DOCKER_IMAGE_TAG'
                    }
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                script {
                    withCredentials([file(credentialsId: 'kubeconfig', variable: 'KUBECONFIG')]) {
                        sh 'kubectl --kubeconfig=$KUBECONFIG apply -f k8s/deployment.yaml'
                        sh 'kubectl --kubeconfig=$KUBECONFIG apply -f k8s/secret.yaml'
                    }
                }
            }
        }

        stage('Cleanup') {
            steps {
                script {
                    echo 'Cleaning up Docker system'
                    sh 'docker system prune -f'
                }
            }
        }
    }

    post {
        success {
            echo 'Pipeline executed successfully.'
        }
        failure {
            echo 'Pipeline failed.'
        }
    }
}

