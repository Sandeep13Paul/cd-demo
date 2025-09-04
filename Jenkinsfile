pipeline {
    agent any

    environment {
        DOCKERHUB_REPO = "sandeeppaul/my-repo"
        APP_VERSION = "latest"
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
                withCredentials([file(credentialsId: 'kubeconfig', variable: 'KUBECONFIG')]) {
                    sh """
                        kubectl apply -f deployment.yaml
                        kubectl apply -f service.yaml
                    """
                }
            }
        }
    }

    post {
        success {
            echo "✅ Deployment & Service applied successfully!"
        }
        failure {
            echo "❌ Deployment failed. Check logs."
        }
    }
}