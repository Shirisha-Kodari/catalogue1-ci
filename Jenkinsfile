pipeline {

    agent {
        label 'roboshop'
    }

    environment {
        APP_VERSION = ''
        AWS_REGION = 'us-east-1'
        ACC_ID = '936819548867'
        PROJECT = 'roboshop'
        COMPONENT = 'catalogue'
        ECR_REGISTRY = '936819548867.dkr.ecr.us-east-1.amazonaws.com'
        IMAGE_NAME = 'roboshop/catalogue1'
    }

    options {
        timeout(time: 30, unit: 'MINUTES')
        disableConcurrentBuilds()
    }

    parameters {
        booleanParam(
            name: 'deploy',
            defaultValue: false,
            description: 'Deploy application'
        )
    }

    stages {

        stage('Read package.json') {
            steps {
                script {
                    def packageJSON = readJSON file: 'package.json'
                    APP_VERSION = packageJSON.version

                    echo "Package Version: ${APP_VERSION}"
                }
            }
        }

        stage('Install Dependencies') {
            steps {
                sh '''
                    npm install
                '''
            }
        }

        stage('Unit Testing') {
            steps {
                sh '''
                    echo "Running unit tests"
                '''
            }
        }

        stage('Login to ECR') {
            steps {
                sh '''
                    aws ecr get-login-password --region $AWS_REGION |
                    docker login --username AWS --password-stdin $ECR_REGISTRY
                '''
            }
        }

        stage('Build Docker Image') {
            steps {
                sh '''
                    docker build -t $IMAGE_NAME:$APP_VERSION .
                    docker tag $IMAGE_NAME:$APP_VERSION $ECR_REGISTRY/$IMAGE_NAME:$APP_VERSION
                '''
            }
        }

        stage('Push to ECR') {
            steps {
                sh '''
                    docker push $ECR_REGISTRY/$IMAGE_NAME:$APP_VERSION
                '''
            }
        }
    }

    post {
        always {
            echo 'Cleaning workspace...'
            deleteDir()
        }

        success {
            echo 'Pipeline completed successfully!'
        }

        failure {
            echo 'Pipeline failed!'
        }
    }
}