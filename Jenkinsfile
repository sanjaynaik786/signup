pipeline {
    agent any

    environment {
        AWS_REGION = "ap-south-1"
        ECR_REPO = "599476212618.dkr.ecr.ap-south-1.amazonaws.com/signupapp"
	    AWS_ACCOUNT  = "599476212618" 
        IMAGE_TAG = "${BUILD_NUMBER}"
	    EKS_CLUSTER     = "my-eks-cluster"
        K8S_NAMESPACE   = "default"
        DEPLOYMENT_NAME = "signup-app"
        CONTAINER_NAME  = "signupapp"
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
                        --region ap-south-1 \
                        --name my-eks-cluster

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
