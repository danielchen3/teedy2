pipeline {
    agent any

    environment {
        DEPLOYMENT_NAME = "hello-node"
        CONTAINER_NAME = "docs"
        IMAGE_NAME = "danielchen3/teedy2025_manual"
    }

    stages {
        stage('Start Minikube') {
            steps {
                sh '''
                set -e
                if ! minikube status | grep -q "Running"; then
                    echo "Starting Minikube..."
                    minikube start
                else
                    echo "Minikube already running."
                fi
                '''
            }
        }

        stage('Set Image') {
            steps {
                sh '''
                set -e
                echo "Setting image for deployment..."
                kubectl set image deployment/${DEPLOYMENT_NAME} ${CONTAINER_NAME}=${IMAGE_NAME}
                '''
            }
        }

        stage('Verify Deployment') {
            steps {
                sh '''
                set -e
                echo "Waiting for rollout to complete..."
                kubectl rollout status deployment/${DEPLOYMENT_NAME}
                echo "Current Pods:"
                kubectl get pods
                '''
            }
        }
    }
}
