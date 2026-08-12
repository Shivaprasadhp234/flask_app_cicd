pipeline {
    agent any

    environment {
        DOCKERHUB_CREDS = credentials('dockerhub-creds')
        IMAGE_NAME = "shivaprasadhp2234/flask-cicd-app"
        K8S_MASTER_IP = "192.168.122.126"
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Build Docker Image') {
            steps {
                sh "docker build -t ${IMAGE_NAME}:${BUILD_NUMBER} -t ${IMAGE_NAME}:latest ."
            }
        }

        stage('Push to Docker Hub') {
            steps {
                sh "echo $DOCKERHUB_CREDS_PSW | docker login -u $DOCKERHUB_CREDS_USR --password-stdin"
                sh "docker push ${IMAGE_NAME}:${BUILD_NUMBER}"
                sh "docker push ${IMAGE_NAME}:latest"
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                sshagent(credentials: ['k8s-master-ssh']) {
                    sh """
                        scp -o StrictHostKeyChecking=no k8s/deployment.yaml k8s/service.yaml ubuntu@${K8S_MASTER_IP}:/tmp/
                        ssh -o StrictHostKeyChecking=no ubuntu@${K8S_MASTER_IP} "sudo k3s kubectl apply -f /tmp/deployment.yaml -f /tmp/service.yaml"
                        ssh -o StrictHostKeyChecking=no ubuntu@${K8S_MASTER_IP} "sudo k3s kubectl rollout status deployment/flask-app"
                    """
                }
            }
        }
    }

    post {
        success {
            echo "Deployed successfully! Access app at http://192.168.122.20:30080"
        }
        failure {
            echo "Pipeline failed - check logs above."
        }
    }
}
