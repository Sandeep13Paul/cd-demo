pipeline {
    agent any

    environment {
        DOCKERHUB_REPO = "sandeeppaul/my-repo"
        APP_VERSION = "latest"
        K8S_TOKEN = credentials('k8s-token')
        K8S_SERVER = "https://34.71.202.58"
    }

    stages {
        stage('Checkout from GitHub') {
            steps {
                echo "Cloning GitHub repository..."
                checkout scm
            }
        }

        stage('Pull Docker Image') {
            steps {
                script {
                    echo "Pulling Docker image from DockerHub..."
                    sh """
                        docker pull $DOCKERHUB_REPO:$APP_VERSION
                    """
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                script {
                    sh """
                        kubectl config set-cluster jenkins-cluster --server=$K8S_SERVER --insecure-skip-tls-verify=true
                        kubectl config set-credentials jenkins --token=$K8S_TOKEN
                        kubectl config set-context my-context --cluster=jenkins-cluster --user=jenkins
                        kubectl config use-context my-context

                        kubectl apply -f deployment.yaml
                        kubectl apply -f service.yaml

                        kubectl rollout history deployment my-app
                    """
                }
            }
        }
    }

    post {
        success {
            echo "Deployment & Service applied successfully!"
        }
        failure {
            echo "Deployment failed. Check logs."
        }
    }
}