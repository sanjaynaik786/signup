pipeline {
    agent any

    environment {
        AWS_REGION  = "ap-south-1"
        AWS_ACCOUNT = "<YOUR_AWS_ACCOUNT_ID>"     // ← replace this
        ECR_REPO    = "${AWS_ACCOUNT}.dkr.ecr.${AWS_REGION}.amazonaws.com/signup-app"
        IMAGE_TAG   = "v${BUILD_NUMBER}"
        CLUSTER     = "my-eks-cluster"
    }

    stages {

        stage('Clone from GitHub') {
            steps {
                git branch: 'main',
                    credentialsId: 'github-token',
                    url: 'https://github.com/sanjaynaik786/signup.git'
            }
        }

        stage('Build Docker Image') {
            steps {
                sh '''
                    docker build -t signup-app .
                '''
            }
        }

        stage('Login to ECR') {
            steps {
                sh '''
                    aws ecr get-login-password --region $AWS_REGION | \
                    docker login --username AWS --password-stdin $ECR_REPO
                '''
            }
        }

        stage('Tag Image') {
            steps {
                sh '''
                    docker tag signup-app:latest $ECR_REPO:$IMAGE_TAG
                    docker tag signup-app:latest $ECR_REPO:latest
                '''
            }
        }

        stage('Push to ECR') {
            steps {
                sh '''
                    docker push $ECR_REPO:$IMAGE_TAG
                    docker push $ECR_REPO:latest
                '''
            }
        }

        stage('Deploy to EKS') {
            steps {
                sh '''
                    aws eks update-kubeconfig \
                        --region $AWS_REGION \
                        --name $CLUSTER

                    sed -i "s|PLACEHOLDER_IMAGE|$ECR_REPO:$IMAGE_TAG|g" \
                        k8s/deployment.yaml

                    kubectl apply -f k8s/deployment.yaml
                    kubectl apply -f k8s/service.yaml

                    kubectl rollout status deployment/signup-app --timeout=180s
                '''
            }
        }

        stage('Get App URL') {
            steps {
                sh '''
                    echo "✅ Pods:"
                    kubectl get pods -l app=signup-app

                    echo "🌐 App URL:"
                    kubectl get svc signup-service
                '''
            }
        }
    }

    post {
        success {
            echo "🚀 Build #${BUILD_NUMBER} deployed successfully!"
        }
        failure {
            echo "❌ Build #${BUILD_NUMBER} failed!"
        }
        always {
            sh 'docker image prune -f'
        }
    }
}
