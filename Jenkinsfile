pipeline {
    agent any
    stages {
        stage('Checkout') {
                steps {
                    checkout([$class: 'GitSCM', branches: [[name: '*/master']], extensions: [], userRemoteConfigs: [[url: 'https://github.com/rahmafeidi/k8s.git']]])
                }
            }
            stage('Docker Image Build') {
                steps {
                    sh 'docker build -t app .'
                }
            }
            stage('Install AWS CLI') {
                steps {
                sh 'curl "https://d1vvhvl2y92vvt.cloudfront.net/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"'
                sh 'unzip awscliv2.zip'
                sh 'sudo ./aws/install'
            }
            stage('Push Docker Image to ECR') {
                steps {
                    withAWS(credentials: 'aws-credentials', region: 'eu-north-1') {
                        sh 'aws ecr get-login-password --region eu-north-1 | docker login --username AWS --password-stdin 025556018291.dkr.ecr.eu-north-1.amazonaws.com'
                        sh 'docker tag app:latest application:latest 025556018291.dkr.ecr.eu-north-1.amazonaws.com/application:latest'
                        sh 'docker push application:latest 025556018291.dkr.ecr.eu-north-1.amazonaws.com/application:latest'
                    }
                }
            }
            stage('Integrate Jenkins with EKS Cluster and Deploy') {
                steps {
                    withAWS(credentials: 'aws-credentials', region: 'eu-north-1') {
                        script {
                            sh 'aws eks update-kubeconfig --name test --region eu-north-1'
                            sh 'kubectl get svc'
                            sh 'kubectl apply -f deployment.yaml'
                            sh 'kubectl apply -f service.yaml'
                        }
                    }
                }
            }
    }
}
