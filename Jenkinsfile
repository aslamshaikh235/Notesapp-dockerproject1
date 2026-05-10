pipeline {
    agent any

    environment {
        AWS_ACCOUNT_ID = "531968433537"
        AWS_DEFAULT_REGION = "us-east-1"
        IMAGE_REPO_NAME = "notes-app"
        IMAGE_TAG = "latest"
    }

    stages {

        stage('Clone Code') {
            steps {
                git branch: 'main',
                url: 'https://github.com/aslamshaikh235/Notesapp-dockerproject1.git'
            }
        }

        stage('SonarQube Scan') {
            steps {
                withCredentials([string(credentialsId: 'sonar-token', variable: 'SONAR_TOKEN')]) {
                    
                    sh '''
                    docker run --rm \
                    -v $(pwd):/usr/src \
                    -e SONAR_HOST_URL="http://100.52.171.109:9000" \
                    -e SONAR_TOKEN="$SONAR_TOKEN" \
                    sonarsource/sonar-scanner-cli \
                    -Dsonar.projectKey=notes-app \
                    -Dsonar.sources=. \
                    -Dsonar.host.url=http://100.52.171.109:9000 \
                    -Dsonar.token=$SONAR_TOKEN
                    '''
                }
            }
        }

        stage('Build Docker Image') {
            steps {
                sh 'docker build -t notes-app .'
            }
        }

        stage('Tag Docker Image') {
            steps {
                sh '''
                docker tag notes-app:latest \
                $AWS_ACCOUNT_ID.dkr.ecr.$AWS_DEFAULT_REGION.amazonaws.com/$IMAGE_REPO_NAME:$IMAGE_TAG
                '''
            }
        }

        stage('Login to ECR') {
            steps {
                withCredentials([[
                    $class: 'AmazonWebServicesCredentialsBinding',
                    credentialsId: 'aws-creds'
                ]]) {

                    sh '''
                    aws ecr get-login-password --region $AWS_DEFAULT_REGION | \
                    docker login --username AWS --password-stdin \
                    $AWS_ACCOUNT_ID.dkr.ecr.$AWS_DEFAULT_REGION.amazonaws.com
                    '''
                }
            }
        }

        stage('Push Image to ECR') {
            steps {
                sh '''
                docker push \
                $AWS_ACCOUNT_ID.dkr.ecr.$AWS_DEFAULT_REGION.amazonaws.com/$IMAGE_REPO_NAME:$IMAGE_TAG
                '''
            }
        }
    }
}
