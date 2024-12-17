pipeline {
    agent any

    environment {
        DOCKER_IMAGE_NAME = "lisandrodev/simple-nodejs"
        IMAGE_TAG = "simple-nodejs"
        AWS_REGION = 'us-east-1' // Set your AWS region
        ECR_REPO_NAME = 'simplejs-app' // Set your ECR repo name
        ACCOUNT_ID='841162685303'
    }

    stages {
        stage('Add User to Docker Group') {
            steps {
                script {
                    // Add user to 'docker' group
                    sh '''
                    sudo usermod -aG docker $(whoami)
                    echo "User added to docker group"
                    
                    
                    # Reload group membership for the docker group
                    sg docker -c "echo 'Docker group reloaded successfully'"
                    '''
                }
            }
        }
        stage('Checkout') {
            steps {
                git branch: "${env.BRANCH_NAME}", url: 'https://github.com/LisandroLuna/simple-nodejs.git'
            }
        }

        stage('Login to AWS ECR') {
            steps { 
                withCredentials([[
                    $class: 'AmazonWebServicesCredentialsBinding',
                    credentialsId: 'aws-credentials-id' // Use your AWS credentials ID
                ]]) {
                    sh '''
                    aws --version
                    aws ecr get-login-password --region $AWS_REGION | \
                    docker login --username AWS --password-stdin $(aws sts get-caller-identity --query Account --output text).dkr.ecr.$AWS_REGION.amazonaws.com
                    '''
                }
            }
        }

        stage('Build Docker Image') {
            steps {
                sh "docker build -t ${ECR_REPO_NAME}:${env.BRANCH_NAME}-${env.BUILD_NUMBER} ."
            }
        }

        stage('Test') {
            steps {
                script {
                    sh "docker run --rm ${ECR_REPO_NAME}:${env.BRANCH_NAME}-${env.BUILD_NUMBER} npm test"
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
                    sh "sudodocker tag ${ECR_REPO_NAME}:${IMAGE_TAG} ${ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com/${ECR_REPO_NAME}:${env.BRANCH_NAME}-${env.BUILD_NUMBER}"
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
                    sh "docker push ${ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com/${ECR_REPO_NAME}:${env.BRANCH_NAME}-${env.BUILD_NUMBER}"
                }
            }
        }
    }

    post {
        always {
            script {
                try {
                    sh "docker rmi ${ECR_REPO_NAME}:${IMAGE_TAG}-${env.BUILD_NUMBER}"
                } catch (Exception e) {
                    echo 'Failed to remove Docker image.'
                }
            }
        }
    }
}
