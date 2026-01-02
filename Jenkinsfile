pipeline {
    agent any

    environment {
        AWS_REGION = "ap-south-1"
        ECR_REPO = "952573859059.dkr.ecr.ap-south-1.amazonaws.com/signupapp"
        IMAGE_TAG = "${BUILD_NUMBER}"
		EKS_CLUSTER     = "my-eks-cluster"
        K8S_NAMESPACE   = "default"
        DEPLOYMENT_NAME = "signup-app"
        CONTAINER_NAME  = "signupapp"
    }

    stages {

        stage('Checkout Code') {
            steps {
				checkout scm
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
                withCredentials([[
                    $class: 'AmazonWebServicesCredentialsBinding',
                    credentialsId: 'aws-jenkins'
                ]]) {
                    sh '''
                      aws ecr get-login-password --region $AWS_REGION | \
                      docker login --username AWS --password-stdin $ECR_REPO
                    '''
                }
            }
        }

        stage('Tag Image') {
            steps {
                sh '''
                  docker tag signup-app:latest $ECR_REPO:$IMAGE_TAG
                '''
            }
        }

        stage('Push Image to ECR') {
            steps {
                sh '''
                  docker push $ECR_REPO:$IMAGE_TAG
                '''
            }
        }
		
		stage('Update kubeconfig') {
            steps {
                sh """
                aws eks update-kubeconfig \
                --region ${AWS_REGION} \
                --name ${EKS_CLUSTER}
                """
            }
        }

        stage('Deploy to EKS') {
            steps {
                sh """
                kubectl set image deployment/${DEPLOYMENT_NAME} \
                ${CONTAINER_NAME}=${ECR_REPO}:${IMAGE_TAG} \
                -n ${K8S_NAMESPACE}
                """
            }
        }

        stage('Verify Rollout') {
            steps {
                sh """
                kubectl rollout status deployment/${DEPLOYMENT_NAME} \
                -n ${K8S_NAMESPACE}
                """
            }
        }
    }

    post {
        success {
            echo "✅ Deployment successful with zero downtime"
        }
        failure {
            echo "❌ Deployment failed"
        }
    }
}
