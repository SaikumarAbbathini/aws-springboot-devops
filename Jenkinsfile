pipeline {
    agent any

    environment {
        IMAGE_NAME = 'saikumar990890/aws-springboot-devops'
        IMAGE_TAG = "${BUILD_NUMBER}"
        KUBECONFIG = '/var/lib/jenkins/.kube/config'
    }

    stages {
        stage('Docker Build and Push') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'DockerHub',
                    usernameVariable: 'DOCKERHUB_USERNAME',
                    passwordVariable: 'DOCKERHUB_PASSWORD'
                )]) {
                    sh '''
                        set -eu
                        printf '%s' "$DOCKERHUB_PASSWORD" |
                            docker login --username "$DOCKERHUB_USERNAME" --password-stdin
                        docker build --tag "$IMAGE_NAME:$IMAGE_TAG" .
                        docker push "$IMAGE_NAME:$IMAGE_TAG"
                        docker logout
                    '''
                }
            }
        }

        stage('Deploy to local Kubernetes') {
            steps {
                sh '''
                    set -eu
                    kubectl apply -f k8s/deployment.yaml
                    kubectl apply -f k8s/service.yaml
                    kubectl set image deployment/aws-devops-demo aws-devops-demo="$IMAGE_NAME:$IMAGE_TAG"
                    kubectl rollout status deployment/aws-devops-demo
                '''
            }
        }
    }
}
