pipeline {
    agent any

    stages {
        stage('Clone Repository') {
            steps {
                script {
                    // Clone the repository using a proper credential ID (adjust as necessary)
                    git credentialsId: 'github-credentials', url: 'https://github.com/your-repo.git'
                }
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    // Build Docker image
                    docker.build("devsecops-nodejs-app:${env.BUILD_ID}")
                }
            }
        }

        stage('Security Scan') {
            steps {
                script {
                    // Run Trivy to scan the image for vulnerabilities
                    sh 'docker run --rm -v /var/run/docker.sock:/var/run/docker.sock aquasec/trivy:latest image devsecops-nodejs-app:${env.BUILD_ID}'
                }
            }
        }

        stage('Push Image to DockerHub') {
            steps {
                script {
                    // Push the image to Docker Hub using configured credentials
                    docker.withRegistry('https://index.docker.io/v1/', 'dockerhub-credentials') {
                        docker.image("devsecops-nodejs-app:${env.BUILD_ID}").push()
                    }
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                script {
                    // Deploy the image to Kubernetes using the kubeconfig
                    kubernetesDeploy(kubeconfigId: 'kubeconfig', configs: 'k8s/deployment.yaml', enableConfigSubstitution: true)
                }
            }
        }
    }

    post {
        always {
            // Clean up any Docker images after the pipeline run
            cleanWs()
        }
    }
}

