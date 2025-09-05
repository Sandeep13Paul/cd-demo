pipeline {
    agent any

    environment {
        DOCKERHUB_REPO = "sandeeppaul/my-repo"
        K8S_TOKEN = credentials('k8s-token')
        K8S_SERVER = "https://34.71.202.58"
    }

    parameters {
        string(name: 'APP_VERSION', defaultValue: 'latest', description: 'Image version to deploy')
        string(name: 'GIT_COMMIT', defaultValue: 'unknown', description: 'Git commit hash')
    }

    stages {
        stage('Checkout from GitHub') {
            steps {
                echo "Cloning GitHub repository..."
                checkout scm
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
                        kubectl set image deployment/my-app my-app-container=$DOCKERHUB_REPO:$APP_VERSION --record
                        kubectl annotate deployment my-app kubernetes.io/change-cause="Deployed via Jenkins build ${BUILD_NUMBER} (commit: ${GIT_COMMIT})" --overwrite
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