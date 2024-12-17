pipeline {
    agent any

    environment {
        AWS_REGION = 'us-east-1' // Set your AWS region
        ECR_REPO_NAME = 'simplejs-app' // Set your ECR repo name
        ACCOUNT_ID='841162685303'
    }

    stages {
        stage('Checkout') {
            steps {
                git branch: "${env.BRANCH_NAME}", url: 'https://github.com/LisandroLuna/simple-nodejs.git'
            }
        }

        stage('Build Docker Image') {
            steps {
                sh "sudo docker build -t ${ECR_REPO_NAME}:${env.BRANCH_NAME}-${env.BUILD_NUMBER} ."
            }
        }

        stage('Test') {
            steps {
                script {
                    sh "sudo docker run --rm ${ECR_REPO_NAME}:${env.BRANCH_NAME}-${env.BUILD_NUMBER} npm test"
                }
            }
        }

        stage('Tag Docker Image') {
            steps {
                withCredentials([[
                    $class: 'AmazonWebServicesCredentialsBinding',
                    credentialsId: 'aws-credentials-id' // Use your AWS credentials ID
                ]]) 
                {  
                    sh "sudo docker tag ${ECR_REPO_NAME}:${env.BRANCH_NAME}-${env.BUILD_NUMBER} ${ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com/${ECR_REPO_NAME}:${env.BRANCH_NAME}-${env.BUILD_NUMBER}"
                }
            }
        }
        stage('Push to AWS ECR') {
            steps {
                withCredentials([[
                    $class: 'AmazonWebServicesCredentialsBinding',
                    credentialsId: 'aws-credentials-id' // Use your AWS credentials ID
                ]]) 
                {
                    // Log in to AWS ECR
                    sh """
                        aws ecr get-login-password --region ${AWS_REGION} | \
                        sudo docker login --username AWS --password-stdin ${ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com
                    """

                    // Push the Docker image
                    sh "sudo docker push ${ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com/${ECR_REPO_NAME}:${env.BRANCH_NAME}-${env.BUILD_NUMBER}"        
                }
            }
        }
    }

    post {
        always {
            script {
                try {
                    sh "sudo docker rmi ${ECR_REPO_NAME}:${IMAGE_TAG}-${env.BUILD_NUMBER}"
                } catch (Exception e) {
                    echo 'Failed to remove Docker image.'
                }
            }
        }
    }
}
