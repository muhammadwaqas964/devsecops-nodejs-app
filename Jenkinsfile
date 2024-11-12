pipeline {
    agent any

    stages {
        stage('Clone Repository') {
            steps {
                git 'https://github.com/muhammadwaqas964/devsecops-nodejs-app.git'
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    docker.build("devsecops-nodejs-app:${env.BUILD_ID}")
                }
            }
        }

        stage('Security Scan') {
            steps {
                sh 'docker run --rm -v /var/run/docker.sock:/var/run/docker.sock aquasec/trivy:latest image devsecops-nodejs-app:${env.BUILD_ID}'
            }
        }

        stage('Push Image to DockerHub') {
            steps {
                script {
                    docker.withRegistry('https://index.docker.io/v1/', 'dockerhub-credentials') {
                        docker.image("devsecops-nodejs-app:${env.BUILD_ID}").push()
                    }
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                kubernetesDeploy(kubeconfigId: 'kubeconfig', configs: 'k8s/deployment.yaml', enableConfigSubstitution: true)
            }
        }
    }
}

