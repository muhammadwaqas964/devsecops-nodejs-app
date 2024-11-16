pipeline {
    agent any

    environment {
        DOCKER_IMAGE_NAME = "muhammadwaqas366/devsecops-nodejs-app"
        DOCKER_IMAGE_TAG = "latest"
        DOCKER_CREDENTIALS = "dockerhub-credentials"
        GITHUB_CREDENTIALS = "Github_AWS_PEOJECT"
        K8S_NAMESPACE = "default"
        KUBECONFIG_PATH = "/home/jenkins/.kube/config"
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
                    withCredentials([file(credentialsId: 'kubeconfig', variable: 'KUBECONFIG'),
                                     [$class: 'AmazonWebServicesCredentialsBinding', credentialsId: 'aws-credentials']]) {
                        // Configure AWS CLI with the provided credentials
                        sh 'aws configure set aws_access_key_id $AWS_ACCESS_KEY_ID'
                        sh 'aws configure set aws_secret_access_key $AWS_SECRET_ACCESS_KEY'
                        sh 'aws configure set region us-east-1'
                        
                        // Apply the Kubernetes manifests
                        sh 'kubectl --kubeconfig=$KUBECONFIG_PATH apply -f k8s/deployment.yaml --validate=false'
                        sh 'kubectl --kubeconfig=$KUBECONFIG_PATH apply -f k8s/secret.yaml --validate=false'
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

