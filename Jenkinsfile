pipeline {
    agent any

    environment {
        IMAGE_NAME = 'saikumar990890/aws-springboot-demo'
        IMAGE_TAG = "${BUILD_NUMBER}"
    }

    stages {
        stage('Checkout develop') {
            steps {
                git branch: 'develop',
                    url: 'https://github.com/SaikumarAbbathini/aws-springboot-devops.git'
            }
        }

        stage('Docker Build and Push') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'DockerHub',
                    usernameVariable: 'DOCKERHUB_USERNAME',
                    passwordVariable: 'DOCKERHUB_PASSWORD'
                )]) {
                    powershell '''
                        $ErrorActionPreference = 'Stop'
                        $env:DOCKERHUB_PASSWORD | docker login --username $env:DOCKERHUB_USERNAME --password-stdin
                        if ($LASTEXITCODE -ne 0) { exit $LASTEXITCODE }
                        docker build --tag "$($env:IMAGE_NAME):$($env:IMAGE_TAG)" .
                        if ($LASTEXITCODE -ne 0) { exit $LASTEXITCODE }
                        docker push "$($env:IMAGE_NAME):$($env:IMAGE_TAG)"
                        if ($LASTEXITCODE -ne 0) { exit $LASTEXITCODE }
                        docker logout
                    '''
                }
            }
        }

        stage('Deploy to local Minikube') {
            steps {
                powershell '''
                    $ErrorActionPreference = 'Stop'
                    kubectl apply -f k8s/deployment.yaml
                    if ($LASTEXITCODE -ne 0) { exit $LASTEXITCODE }
                    kubectl apply -f k8s/service.yaml
                    if ($LASTEXITCODE -ne 0) { exit $LASTEXITCODE }
                    kubectl set image deployment/aws-devops-demo aws-devops-demo="$($env:IMAGE_NAME):$($env:IMAGE_TAG)"
                    if ($LASTEXITCODE -ne 0) { exit $LASTEXITCODE }
                    kubectl rollout status deployment/aws-devops-demo
                    if ($LASTEXITCODE -ne 0) { exit $LASTEXITCODE }
                    '''
            }
        }
    }
}
