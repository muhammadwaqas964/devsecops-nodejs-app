pipeline {
    agent any

    environment {
        DOCKER_IMAGE_NAME = "muhammadwaqas366/devsecops-nodejs-app"  // DockerHub image name
        DOCKER_IMAGE_TAG = "latest"  // Docker image tag
        DOCKER_CREDENTIALS = "dockerhub-credentials"  // Jenkins credentials ID for DockerHub
        GITHUB_CREDENTIALS = "Github_AWS_PEOJECT"  // Jenkins credentials ID for GitHub
        K8S_NAMESPACE = "default"  // Kubernetes namespace (you can change this as needed)
        KUBECONFIG_PATH = "/home/jenkins/.kube/config"  // Path to kubeconfig file
        AWS_CREDENTIALS = "aws-credentials"  // AWS credentials if needed
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
                    // Log in to DockerHub using Jenkins credentials
                    withCredentials([usernamePassword(credentialsId: DOCKER_CREDENTIALS, passwordVariable: 'DOCKER_PASSWORD', usernameVariable: 'DOCKER_USERNAME')]) {
                        sh 'echo $DOCKER_PASSWORD | docker login --username $DOCKER_USERNAME --password-stdin'
                    }
                    // Build the Docker image for the Node.js app
                    sh 'docker build -t $DOCKER_IMAGE_NAME:$DOCKER_IMAGE_TAG .'
                }
            }
        }

        stage('Security Scan') {
            steps {
                script {
                    // Example security scan with Trivy or similar tool (uncomment and configure if needed)
                    echo "Running security scan on Docker image"
                    // Example: sh 'trivy image $DOCKER_IMAGE_NAME:$DOCKER_IMAGE_TAG'
                }
            }
        }

        stage('Push Image to DockerHub') {
            steps {
                script {
                    // Push the built image to DockerHub
                    withCredentials([usernamePassword(credentialsId: DOCKER_CREDENTIALS, passwordVariable: 'DOCKER_PASSWORD', usernameVariable: 'DOCKER_USERNAME')]) {
                        sh 'docker push $DOCKER_IMAGE_NAME:$DOCKER_IMAGE_TAG'
                    }
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                script {
                    // Deploy to Kubernetes using the kubeconfig credential
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
                    // Clean up Docker resources to free up space
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

